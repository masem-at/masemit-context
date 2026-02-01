---
stepsCompleted: [1, 2, 3]
inputDocuments:
  - docs/prd.md
  - docs/architecture.md
  - docs/ux-design-specification.md
  - docs/design/dao-rankings-leaderboard-ux-validation.md
  - docs/project_context.md
lastStep: 3
status: 'stories-complete'
totalEpics: 10
totalRequirements: 228
totalStories: 51
---

# ChainSights DAO Rankings Leaderboard - Epic Breakdown

## Overview

This document provides the complete epic and story breakdown for the DAO Rankings Leaderboard feature, decomposing the requirements from the PRD, Architecture, and UX validation into implementable stories.

**Scope:** DAO Rankings Leaderboard (public-facing governance scoring and ranking system)

**Project Context:** Next.js 14 App Router, TypeScript strict mode, Drizzle ORM (PostgreSQL), Server Components by default, Vercel serverless deployment

## Requirements Inventory

### Functional Requirements (from PRD)

**FR1-FR12: Governance Scoring & Calculation**
- FR1: System can calculate GVS using weighted formula: (HPR × 0.35) + (DEI × 0.25) + (PDI × 0.20) + (GPI × 0.20)
- FR2: System can calculate Human Participation Rate (HPR) component
- FR3: System can calculate Delegate Engagement Index (DEI) component
- FR4: System can calculate Power Dynamics Index (PDI) component
- FR5: System can calculate Grassroots Participation Index (GPI) component
- FR6: System can assign confidence scores based on data completeness (High/Medium/Low)
- FR7: System can exclude DAOs when data confidence is below 50%
- FR8: System can fetch governance data from Snapshot.org API
- FR9: System can fetch token distribution data from Ethereum blockchain
- FR10: System can store historical GVS snapshots with timestamps
- FR11: System can calculate week-over-week GVS deltas (+/- change values)
- FR12: System can execute automated weekly GVS recalculation job (Sunday midnight UTC)

**FR13-FR26: Leaderboard Display & Navigation**
- FR13: Users can view ranked list of DAOs sorted by GVS score (highest to lowest)
- FR14: Users can see each DAO's rank, name, GVS score, score label, category, and badge type
- FR15: Users can see score labels with color coding (Vital: 80-100, Stable: 60-79, Concerning: 40-59, Critical: 0-39)
- FR16: Users can see badge types indicating analysis source (Featured Analysis vs Customer Report)
- FR17: Users can see week-over-week change indicators (+/- score change with directional arrows and color coding)
- FR18: Users can filter leaderboard by DAO category (All/DeFi/Infrastructure/Public Goods)
- FR19: Users can sort leaderboard by different columns (rank, score, change, category)
- FR20: Users can expand DAO entries to view component score breakdown (HPR, DEI, PDI, GPI with individual scores)
- FR21: Users can view "last updated" timestamp showing when GVS scores were last calculated
- FR22: Users can navigate to individual DAO profile pages (future enhancement - Phase 3)
- FR23: Users can access leaderboard on mobile devices with vertical scrolling table layout
- FR24: Users can access leaderboard on desktop with full-width table showing all columns
- FR25: Users can navigate leaderboard using keyboard (tab through rows, enter to expand)
- FR26: Screen reader users can access leaderboard with semantic HTML structure and ARIA labels

**FR27-FR36: Methodology Transparency**
- FR27: Users can view methodology page explaining GVS calculation approach
- FR28: Users can see detailed explanation of each GVS component (HPR, DEI, PDI, GPI)
- FR29: Users can see component weighting breakdown (percentages for each component)
- FR30: Users can view data sources list (Snapshot.org, Ethereum RPC providers)
- FR31: Users can see update frequency disclosure (weekly recalculation, Sunday midnight UTC)
- FR32: Users can view legal disclaimers (opinions not facts, no investment advice)
- FR33: Users can see opt-out policy (24-hour removal guarantee)
- FR34: Users can see confidence scoring methodology (High/Medium/Low criteria)
- FR35: Users can view contact information for questions/disputes
- FR36: Users can see methodology version number and last updated date

**FR37-FR49: Customer Journey Management**
- FR37: Users can view "Get Report" CTA in expanded DAO rows
- FR38: Users can click CTA to navigate to report ordering flow
- FR39: Users can see pricing information on CTA button (€49-149 range)
- FR40: Users can distinguish between Featured Analysis (ChainSights-commissioned) and Customer Report badges
- FR41: DAOs with Customer Reports can see "Customer Report" badge on leaderboard
- FR42: DAOs can opt-in to public listing after purchasing report
- FR43: DAOs can opt-out from leaderboard visibility via opt-out form
- FR44: Users can access opt-out form at `/rankings/opt-out`
- FR45: Users can submit opt-out request with DAO name, contact email, and reason
- FR46: System sends confirmation email to opt-out requester
- FR47: Admin can process opt-out requests within 24 hours
- FR48: System removes opted-out DAOs from public leaderboard
- FR49: System retains historical data for opted-out DAOs (internal only, not public)

**FR50-FR59: Administrative Operations**
- FR50: Admin can manually trigger GVS recalculation for specific DAOs
- FR51: Admin can manually trigger GVS recalculation for all DAOs
- FR52: Admin can view job execution logs (start time, duration, status)
- FR53: Admin can view failed job error messages
- FR54: Admin can manually add DAOs to Featured Analysis list
- FR55: Admin can manually remove DAOs from Featured Analysis list
- FR56: Admin can override confidence scores for specific DAOs
- FR57: Admin can pause weekly job execution temporarily
- FR58: Admin can view pending opt-out requests
- FR59: Admin can mark opt-out requests as processed

**FR60-FR66: Legal & Compliance**
- FR60: System validates all user inputs to prevent XSS attacks
- FR61: System validates all user inputs to prevent SQL injection
- FR62: System displays legal disclaimers on methodology page
- FR63: System displays privacy policy link in footer
- FR64: System complies with GDPR right to erasure (24-hour opt-out)
- FR65: System minimizes data collection (no personal data beyond consented emails)
- FR66: System logs all opt-out requests for audit trail

**FR67-FR74: Social & Marketing Integration**
- FR67: Users can share leaderboard page on social media (Twitter/X, LinkedIn)
- FR68: Users can share specific DAO entries with pre-filled text
- FR69: System generates Open Graph tags for social media previews
- FR70: System generates Twitter Card tags for rich previews
- FR71: Expanded DAO rows display "Get Report" CTA with pricing
- FR72: Expanded DAO rows display "Share on X" button
- FR73: System tracks CTA click-through rates
- FR74: System tracks social share button clicks

**FR75-FR81: Performance & Reliability**
- FR75: Leaderboard page loads with LCP < 2.0s (p95)
- FR76: Leaderboard page achieves FCP < 1.5s (p95)
- FR77: System implements ISR caching (1-hour revalidation)
- FR78: System gracefully handles Snapshot API failures (serve cached data)
- FR79: System gracefully handles Ethereum RPC failures (skip on-chain verification)
- FR80: System retries failed API calls with exponential backoff (3 attempts)
- FR81: System implements rate limiting (100 requests/minute per IP)

**FR82-FR86: Search Engine Optimization**
- FR82: System generates sitemap.xml including all public pages
- FR83: System implements robots.txt allowing crawlers (disallow /admin)
- FR84: System adds canonical URLs to all pages
- FR85: System implements structured data (JSON-LD) for leaderboard
- FR86: System implements structured data (JSON-LD) for methodology page

**FR87-FR93: Analytics & Monitoring**
- FR87: System tracks Core Web Vitals (FCP, LCP, TTI, CLS, TBT)
- FR88: System tracks page views on leaderboard and methodology pages
- FR89: System tracks CTA conversion rates (clicks → report orders)
- FR90: System tracks social share rates (shares per DAO listing)
- FR91: System tracks opt-out rate (opt-outs / total DAOs ranked)
- FR92: System tracks repeat visitor rate (returning visitors percentage)
- FR93: System sends alerts when LCP increases >20% week-over-week

### Non-Functional Requirements (from PRD)

**NFR-PERF: Performance**
- NFR-PERF-1: Page Load - FCP < 1.5s (p95), < 2.5s (p99) - Lighthouse CI
- NFR-PERF-2: Page Load - LCP < 2.0s (p95), < 3.0s (p99) - Core Web Vitals
- NFR-PERF-3: Page Load - TTI < 3.0s (p95), < 4.5s (p99) - Lighthouse audit
- NFR-PERF-4: Page Load - CLS < 0.1 (p95), < 0.2 (p99) - Visual stability
- NFR-PERF-5: Page Load - TBT < 200ms (p95), < 400ms (p99) - Main thread responsiveness
- NFR-PERF-6: Backend - GVS calculation per DAO < 5 minutes (target), < 10 minutes (acceptable), timeout after 10 min
- NFR-PERF-7: Backend - Weekly job for 50 DAOs < 2 hours (target), < 4 hours (acceptable)
- NFR-PERF-8: Backend - API response time < 500ms (p95), < 1s (p99), error if > 5s
- NFR-PERF-9: Backend - Database query < 100ms (p95), < 500ms (p99), timeout after 5s
- NFR-PERF-10: Asset Budgets - JavaScript bundle < 150KB gzipped (fail build if exceeded)
- NFR-PERF-11: Asset Budgets - CSS < 30KB gzipped
- NFR-PERF-12: Asset Budgets - Fonts < 50KB WOFF2 (Montserrat + Inter subsets)
- NFR-PERF-13: Asset Budgets - Images < 500KB total per page
- NFR-PERF-14: Asset Budgets - API response payload < 100KB per query

**NFR-SEC: Security**
- NFR-SEC-1: Encryption - All traffic served over HTTPS with HSTS header (max-age=31536000)
- NFR-SEC-2: Encryption - Database encryption enabled via Neon managed Postgres
- NFR-SEC-3: Input Validation - All user inputs sanitized to prevent XSS, SQL injection, command injection
- NFR-SEC-4: Rate Limiting - 100 requests per minute per IP via Vercel Edge Middleware
- NFR-SEC-5: Content Security Policy - Restrict script sources to prevent code injection attacks
- NFR-SEC-6: CORS - API endpoints restricted to same-origin only (no cross-origin until Phase 3)
- NFR-SEC-7: Admin Access - Protected routes require authentication (Vercel basic auth or session-based)
- NFR-SEC-8: Public Access - No user authentication required for public leaderboard (barrier-free)
- NFR-SEC-9: GDPR - Data minimization (no personal data beyond consented emails)
- NFR-SEC-10: GDPR - Right to erasure (24-hour opt-out removal from rankings)
- NFR-SEC-11: GDPR - Data retention (email addresses deleted 90 days after report delivery unless marketing opt-in)
- NFR-SEC-12: GDPR - Cookie consent (analytics cookies only with disclosure)
- NFR-SEC-13: GDPR - Privacy policy linked from footer explaining data collection and usage
- NFR-SEC-14: Vulnerability Management - Automated Dependabot alerts for npm package vulnerabilities
- NFR-SEC-15: Security Headers - X-Content-Type-Options, X-Frame-Options, X-XSS-Protection configured
- NFR-SEC-16: Error Handling - No sensitive data leaked in error messages (stack traces hidden in production)

**NFR-SCALE: Scalability**
- NFR-SCALE-1: User Growth - Week 1: 6 DAOs, 500 visitors/month, 0-2 reports/month - MVP deployment sufficient
- NFR-SCALE-2: User Growth - Month 1: 15-20 DAOs, 2,000 visitors/month, 10 reports/month - Current architecture sufficient
- NFR-SCALE-3: User Growth - Month 3: 30+ DAOs, 5,000 visitors/month, 30 reports/month - Requires cache optimization, database indexing
- NFR-SCALE-4: User Growth - Month 6: 50-75 DAOs, 10,000 visitors/month, 50 reports/month - Consider Redis, optimize weekly job
- NFR-SCALE-5: User Growth - Month 12: 100+ DAOs, 25,000 visitors/month, 100+ reports/month - Evaluate dedicated API server, CDN
- NFR-SCALE-6: Horizontal Scalability - Vercel serverless automatic horizontal scaling (no manual intervention)
- NFR-SCALE-7: Horizontal Scalability - Neon Postgres serverless with automatic connection pooling
- NFR-SCALE-8: Horizontal Scalability - Redis cache added when API load exceeds 10,000 requests/day (Month 6+)
- NFR-SCALE-9: Vertical Scalability - GVS calculation parallel processing (10 DAOs simultaneously instead of sequential)
- NFR-SCALE-10: Vertical Scalability - Database queries indexed on frequently queried columns (score, category, rank)
- NFR-SCALE-11: Vertical Scalability - Asset delivery via Next.js Image optimization with automatic WebP conversion
- NFR-SCALE-12: Degradation Strategy - If load exceeds capacity: Increase cache TTL (1-hour → 6-hour ISR revalidation)
- NFR-SCALE-13: Degradation Strategy - If load exceeds capacity: Paginate leaderboard (Top 50, then "Load More" for rest)
- NFR-SCALE-14: Degradation Strategy - If load exceeds capacity: Pause weekly job (manual recalculation only for critical DAOs)
- NFR-SCALE-15: Capacity Alerts - Alert if database connection pool > 80% utilization
- NFR-SCALE-16: Capacity Alerts - Alert if API response time p95 exceeds 1 second for 5 minutes
- NFR-SCALE-17: Capacity Alerts - Alert if weekly job duration increases > 50% week-over-week

**NFR-ACCESS: Accessibility (WCAG 2.1 Level AA)**
- NFR-ACCESS-1: Visual - Color contrast minimum 4.5:1 for body text, 7:1 for headings (WebAIM Contrast Checker)
- NFR-ACCESS-2: Visual - Visible 2px aqua outline on all interactive elements (focus indicators)
- NFR-ACCESS-3: Visual - Layout remains functional at 200% browser zoom (text resizing)
- NFR-ACCESS-4: Visual - Change indicators use arrows (↗/↘/→) + color (no information from color alone)
- NFR-ACCESS-5: Keyboard - Full keyboard navigation (tab through all interactive elements in logical order)
- NFR-ACCESS-6: Keyboard - Skip links ("Skip to main content" link for screen reader users)
- NFR-ACCESS-7: Keyboard - Focus management (focus moves to error messages on form submission failure)
- NFR-ACCESS-8: Keyboard - No keyboard traps (users can navigate away from all components using only keyboard)
- NFR-ACCESS-9: Screen Reader - Semantic HTML (proper `<table>`, `<thead>`, `<tbody>`, `<th scope>` for leaderboard)
- NFR-ACCESS-10: Screen Reader - ARIA labels (aria-label on all icons and non-text controls)
- NFR-ACCESS-11: Screen Reader - ARIA live regions (announce dynamic content changes)
- NFR-ACCESS-12: Screen Reader - Descriptive alt text for all images
- NFR-ACCESS-13: Mobile - Touch targets minimum 44×44px for all buttons and links (WCAG 2.5.5)
- NFR-ACCESS-14: Mobile - Layout works in both portrait and landscape (orientation)
- NFR-ACCESS-15: Mobile - Respect `prefers-reduced-motion` media query (disable animations for motion sensitivity)
- NFR-ACCESS-16: Testing - Automated testing via axe-core in Playwright tests (catches 60% of issues)
- NFR-ACCESS-17: Testing - Manual testing with NVDA (Windows) + VoiceOver (macOS/iOS) screen readers
- NFR-ACCESS-18: Testing - Accessibility statement public page at `/accessibility` with contact for issues

**NFR-REL: Reliability & Availability**
- NFR-REL-1: Uptime - Public pages 99.5% uptime (allows ~3.6 hours downtime per month)
- NFR-REL-2: Uptime - Weekly job 100% success rate (zero acceptable failures)
- NFR-REL-3: Uptime - Admin dashboard 99% uptime (acceptable - only Mario accesses)
- NFR-REL-4: Data Integrity - Zero data loss for historical GVS snapshots (critical for delta calculations)
- NFR-REL-5: Data Integrity - GVS scores align with manual calculations > 99% of the time (calculation accuracy)
- NFR-REL-6: Data Integrity - Component scores (HPR+DEI+PDI+GPI) mathematically sum to GVS (100% accuracy)
- NFR-REL-7: Error Recovery - Snapshot API failure: Serve cached data with "Last updated [timestamp]" notice
- NFR-REL-8: Error Recovery - Database failure: Display maintenance page within 30 seconds, alert admin immediately
- NFR-REL-9: Error Recovery - Weekly job failure: Retry once automatically, then Slack alert to admin if second failure
- NFR-REL-10: Error Recovery - Payment failure: Customer notified via email, manual invoice fallback
- NFR-REL-11: Monitoring - UptimeRobot pings `/rankings` every 5 minutes
- NFR-REL-12: Monitoring - Sentry captures and alerts on production errors (> 10 errors/hour = immediate alert)
- NFR-REL-13: Monitoring - Cron job logs success/failure, Slack notification on failure
- NFR-REL-14: Monitoring - Alert if LCP increases > 20% week-over-week (performance degradation)
- NFR-REL-15: Backup & Recovery - Neon automatic daily backups (30-day retention)
- NFR-REL-16: Backup & Recovery - Git history allows rollback to any previous deploy
- NFR-REL-17: Backup & Recovery - Vercel 1-click rollback to previous stable deployment

**NFR-MAINT: Maintainability**
- NFR-MAINT-1: Code Quality - 100% TypeScript (no JavaScript files except config)
- NFR-MAINT-2: Code Quality - Type safety with `strict: true` in tsconfig.json, zero `any` types in core logic
- NFR-MAINT-3: Code Quality - ESLint with Next.js recommended rules, zero warnings in production build
- NFR-MAINT-4: Code Quality - Minimum 60% test coverage for GVS calculation engine (critical logic)
- NFR-MAINT-5: Documentation - README.md with setup instructions, architecture overview, deployment guide
- NFR-MAINT-6: Documentation - Inline comments for complex GVS calculations with formula explanations
- NFR-MAINT-7: Documentation - JSDocs for all public functions and components (API documentation)
- NFR-MAINT-8: Documentation - ADRs (Architecture Decision Records) documenting major technical decisions
- NFR-MAINT-9: Dependency Management - npm audit run weekly, critical vulnerabilities patched within 7 days
- NFR-MAINT-10: Dependency Management - Exact versions in package.json (no `^` or `~` for production dependencies)
- NFR-MAINT-11: Dependency Management - Track deprecated npm packages, plan migration before EOL
- NFR-MAINT-12: Refactoring Safety - Unit tests for GVS calculation, integration tests for API routes
- NFR-MAINT-13: Refactoring Safety - Visual regression via Percy.io snapshots for leaderboard page
- NFR-MAINT-14: Refactoring Safety - Every PR gets Vercel preview URL for manual QA before merge

**NFR-INTEG: Integration**
- NFR-INTEG-1: External Systems - Snapshot.org API (99% SLA) - Fallback: Serve cached data, show staleness indicator
- NFR-INTEG-2: External Systems - Ethereum RPC (95% SLA) - Fallback: Skip on-chain verification if unavailable
- NFR-INTEG-3: External Systems - Stripe API (99.9% SLA) - Fallback: Manual invoice fallback
- NFR-INTEG-4: External Systems - Vercel Platform (99.9% SLA) - No fallback (dependency)
- NFR-INTEG-5: External Systems - Neon Postgres (99.95% SLA) - No fallback (dependency)
- NFR-INTEG-6: API Reliability - Retry logic: 3 retries with exponential backoff for transient failures
- NFR-INTEG-7: API Reliability - Timeout handling: 30-second timeout for Snapshot API calls, graceful failure message
- NFR-INTEG-8: API Reliability - Circuit breaker: After 5 consecutive Snapshot API failures, skip external calls for 5 minutes
- NFR-INTEG-9: Data Format Compatibility - Snapshot API: Handle schema changes gracefully (ignore unknown fields, validate required fields)
- NFR-INTEG-10: Data Format Compatibility - Stripe webhooks: Verify webhook signatures to prevent spoofing
- NFR-INTEG-11: Data Format Compatibility - CSV export (Future): UTF-8 encoding, RFC 4180 compliant

**NFR-SEO: SEO & Content Discoverability**
- NFR-SEO-1: Search Engine Optimization - All public pages crawlable by search engines (robots.txt allows)
- NFR-SEO-2: Search Engine Optimization - No `noindex` tags on leaderboard or methodology pages (indexability)
- NFR-SEO-3: Search Engine Optimization - Sitemap.xml generated automatically, includes all public DAO ranking pages
- NFR-SEO-4: Search Engine Optimization - Schema.org ItemList markup for leaderboard (structured data for rich snippets)
- NFR-SEO-5: Search Engine Optimization - Core Web Vitals in "Good" range (Google ranking factor)
- NFR-SEO-6: Social Media Optimization - Open Graph tags (og:title, og:description, og:image) for all public pages
- NFR-SEO-7: Social Media Optimization - Twitter Cards (summary_large_image with leaderboard preview)
- NFR-SEO-8: Social Media Optimization - Shareable URLs (clean, readable URLs like `/rankings`, `/rankings/methodology`)
- NFR-SEO-9: Content Freshness - Weekly updates (new content signal for search engines via GVS recalculations)
- NFR-SEO-10: Content Freshness - `lastmod` in sitemap.xml reflects actual content updates
- NFR-SEO-11: Content Freshness - Canonical URLs prevent duplicate content penalties
- NFR-SEO-12: Target - 1,000+ organic visits per month by Month 6 (baseline: 0 at launch)

### Additional Requirements from Architecture

**ARCH-SETUP: Project Initialization**
- ARCH-SETUP-1: Initialize Next.js 14 project with starter template:
  ```bash
  npx create-next-app@latest . --typescript --tailwind --eslint --app --src-dir --import-alias "@/*" --no-git --use-npm
  ```
- ARCH-SETUP-2: Install core dependencies:
  - Database: drizzle-orm, drizzle-kit, @neondatabase/serverless
  - Web3: wagmi, viem, @walletconnect/ethereum-provider
  - External services: @anthropic-ai/sdk, stripe, resend, @upstash/qstash
  - Testing: vitest, @playwright/test, @axe-core/playwright
- ARCH-SETUP-3: Configure TypeScript strict mode in tsconfig.json
- ARCH-SETUP-4: Configure Drizzle ORM with Neon Postgres connection
- ARCH-SETUP-5: Set up environment variables (.env.example, .env.local)
- ARCH-SETUP-6: Configure Vercel deployment (vercel.json if needed)

**ARCH-TECH: Technical Constraints**
- ARCH-TECH-1: All database identifiers MUST use snake_case (tables, columns, enums)
- ARCH-TECH-2: Use Server Components by default, add 'use client' only when necessary (event handlers, hooks, browser APIs)
- ARCH-TECH-3: Never fetch data in Client Components - do it in Server Components and pass as props
- ARCH-TECH-4: Use Drizzle query builder for ALL database queries (never raw SQL strings)
- ARCH-TECH-5: Validate ALL request bodies with Zod schemas (defined in src/lib/validation/)
- ARCH-TECH-6: Use Next.js Image component for all images (automatic optimization)
- ARCH-TECH-7: Use ISR caching with `export const revalidate = 3600` for 1-hour cache
- ARCH-TECH-8: Never use window.ethereum directly - use wagmi abstractions (for crypto payments in future)

### Additional Requirements from UX Validation

**UX-P0: Critical UX Fixes (Must Fix Before Launch)**

**UX-P0-1: Color-Blind Accessibility Violation (WCAG 1.4.1)**
- Severity: Critical
- Affected: FR26 (Accessibility), NFR-ACCESS-4 (No information from color alone)
- Problem: Score bars use only color to convey meaning (red/yellow/green)
- Fix: Add texture patterns to score bar fills using CSS gradients:
  - Critical (1-3): Diagonal stripes pattern
  - Concerning (4-6): Dotted pattern
  - Stable (7-8): Horizontal lines pattern
  - Vital (9-10): Solid (no pattern)
- Acceptance Criteria:
  - Score bars distinguishable in grayscale
  - Pass axe-core accessibility audit (no color-only violations)
  - Visual regression tests pass with patterns applied
  - Score severity identifiable without color perception

**UX-P0-2: CTA Price Ambiguity**
- Severity: Critical
- Affected: FR71-FR73 (CTA & Conversion)
- Problem: Expanded row shows "Get Full Report — €49" but PRD specifies €49-149 pricing
- Fix Options:
  - Option A: Single tier → "Get Full Report — €49 →"
  - Option B: Tiered pricing → "See Report Options → from €49"
  - Option C: Promotional → "Get Report — €49 Launch Special (reg. €49) →"
- Decision Needed: Mario to confirm actual pricing structure
- Acceptance Criteria:
  - CTA price matches checkout page price
  - No user confusion about pricing in user testing
  - Clear tier differentiation if multiple options exist
  - Conversion tracking shows no price-mismatch bounces

**UX-P1: Important UX Enhancements (MVP+1 Week)**

**UX-P1-1: Mobile Tap Affordance Insufficient**
- Severity: Important
- Affected: FR23 (Mobile accessibility)
- Problem: Mobile spec shows "Tap for details" button, but users expect entire card to be tappable
- Fix: Make entire card tappable with visual chevron indicator:
  - Add chevron (▼) in top-right corner
  - Rotate chevron 180° when expanded (▲)
  - Add hover/active states (slight background change)
  - Animate expansion (smooth height transition)
- Acceptance Criteria:
  - Entire card responds to tap/click
  - Chevron rotates smoothly on expand/collapse
  - Keyboard navigation works (Enter/Space to toggle)
  - Screen reader announces expanded state change
  - Mobile user testing shows > 90% expansion success rate

**UX-P1-2: First-Time User Onboarding Missing**
- Severity: Important
- Affected: User Success Metrics (30-second comprehension goal)
- Problem: No first-visit orientation exists - users may bounce within 15 seconds if confused
- Fix: Add dismissible info banner for first-time visitors (localStorage flag):
  - "New here? This leaderboard ranks DAOs on governance health across 5 criteria..."
  - Two CTAs: "Got it" (dismiss) and "Learn More →" (methodology page)
  - Appears between page header and leaderboard table
- Acceptance Criteria:
  - Banner shows only on first visit
  - localStorage flag persists dismissal
  - "Got it" dismisses banner immediately
  - "Learn More" links to methodology page
  - Banner does not show to returning visitors
  - Bounce rate decreases by > 10% with banner enabled (A/B test)

**UX-P1-3: Top 5 Control Column Cognitive Load**
- Severity: Important
- Affected: FR14 (Score display), User Success (30-sec comprehension)
- Problem: Users must mentally map "52% is concerning" - percentages lack inherent directionality
- Fix: Add visual indicators for Top 5 Control severity:
  - ✓ (green): < 30% - Well-distributed
  - ⚠️ (yellow): 30-50% - Moderate concentration
  - 🔶 (orange): 50-75% - High concentration
  - 🔴 (red): > 75% - Extreme concentration
- Acceptance Criteria:
  - Icons display correctly in all browsers
  - Screen readers announce severity label (aria-label)
  - Color-blind users can distinguish icons by shape
  - User testing shows > 80% correctly identify severity at a glance
  - Icons visible in mobile table layout

**UX-P2: Nice-to-Have Enhancements (Post-MVP)**

**UX-P2-1: Empty State for Low DAO Count**
- Severity: Nice-to-Have
- Value: Converts weakness (low count) into lead-gen opportunity
- Affected: FR18 (Category filtering)
- Enhancement: Show CTA when category filter has < 5 DAOs:
  - "📊 Want to see your DAO ranked here?"
  - "We're actively expanding our {category} coverage. Request a free analysis..."
  - Button: "Request Free Analysis →"
- Acceptance Criteria:
  - Empty state shows when category has < 5 DAOs
  - CTA links to lead capture form
  - Message personalizes by category ("Public Goods coverage")
  - Tracks lead conversions from empty state CTA

**UX-P2-2: Sticky "Last Updated" Header on Scroll**
- Severity: Nice-to-Have
- Value: Builds trust, answers "Is this fresh?" question while scrolling
- Affected: FR21 (Last updated timestamp)
- Enhancement: Sticky header that appears on scroll (after 200px):
  - Shows "DAO Rankings • Updated {time ago}"
  - Expandable dropdown with update schedule details
  - Links to methodology page
- Acceptance Criteria:
  - Banner appears after 200px scroll
  - Banner hides when scrolled back to top
  - Dropdown shows update schedule
  - Does not cover leaderboard content (z-index managed)
  - Mobile-friendly (collapses on small screens)

**UX-P2-3: Historical Trend Sparklines**
- Severity: Nice-to-Have
- Value: HIGH - Transforms static leaderboard into living narrative
- Affected: FR17 (Week-over-week changes), Growth Features (historical trends)
- Enhancement: Add 4-week mini chart (20px tall sparkline) next to change indicator:
  - Shows 4-8 weeks of historical GVS data
  - Color-coded by trend direction (green up, red down)
  - Tooltip on hover with precise values
- Acceptance Criteria:
  - Sparklines render correctly for all DAOs with 4+ weeks history
  - Hover tooltip shows precise weekly values
  - Sparklines accessible to screen readers (data table fallback)
  - Mobile layout includes sparklines (stacked if needed)
  - User engagement increases (measured via time-on-page, repeat visits)

## FR Coverage Map

### Functional Requirements → Epic Mapping

**Epic 0: Project Foundation & Development Setup**
- ARCH-SETUP-1: Initialize Next.js 14 project with starter template
- ARCH-SETUP-2: Install core dependencies (Drizzle, Web3, Stripe, testing)
- ARCH-SETUP-3: Configure TypeScript strict mode
- ARCH-SETUP-4: Configure Drizzle ORM with Neon Postgres
- ARCH-SETUP-5: Set up environment variables
- ARCH-SETUP-6: Configure Vercel deployment
- ARCH-TECH-1 to ARCH-TECH-8: Technical constraints and patterns

**Epic 1: Governance Vitality Score (GVS) Calculation Engine**
- FR1: Calculate GVS using weighted formula
- FR2: Calculate Human Participation Rate (HPR) component
- FR3: Calculate Delegate Engagement Index (DEI) component
- FR4: Calculate Power Dynamics Index (PDI) component
- FR5: Calculate Grassroots Participation Index (GPI) component
- FR6: Assign confidence scores based on data completeness
- FR7: Exclude DAOs when confidence < 50%
- FR8: Fetch governance data from Snapshot.org API
- FR9: Fetch token distribution data from Ethereum blockchain
- FR10: Store historical GVS snapshots with timestamps
- FR11: Calculate week-over-week GVS deltas

**Epic 2: Public Rankings Leaderboard (MVP)**
- FR13: View ranked list sorted by GVS score
- FR14: See DAO rank, name, score, label, category, badge
- FR15: See score labels with color coding
- FR16: See badge types (Featured Analysis vs Customer Report)
- FR17: See week-over-week change indicators
- FR18: Filter leaderboard by DAO category
- FR19: Sort by columns (rank, score, change, category)
- FR20: Expand DAO entries for component breakdown
- FR21: View "last updated" timestamp
- FR22: Navigate to DAO profile pages (future - Phase 3)
- FR23: Access on mobile with vertical scrolling
- FR24: Access on desktop with full-width table
- FR25: Navigate using keyboard
- FR26: Screen reader accessibility
- UX-P0-1: Color-blind accessibility (texture patterns on score bars)
- UX-P0-2: CTA price clarity (€49 confirmed)

**Epic 3: Methodology Transparency & Legal Foundation**
- FR27: View methodology page explaining GVS calculation
- FR28: See detailed explanation of each component
- FR29: See component weighting breakdown
- FR30: View data sources list
- FR31: See update frequency disclosure
- FR32: View legal disclaimers
- FR33: See opt-out policy
- FR34: See confidence scoring methodology
- FR35: View contact information
- FR36: See methodology version and last updated date
- FR60: Validate inputs to prevent XSS
- FR61: Validate inputs to prevent SQL injection
- FR62: Display legal disclaimers
- FR63: Display privacy policy link
- FR65: Minimize data collection
- FR66: Log opt-out requests for audit trail

**Epic 4: Opt-Out Management System**
- FR43: DAOs can opt-out via opt-out form
- FR44: Access opt-out form at /rankings/opt-out
- FR45: Submit opt-out with DAO name, email, reason
- FR46: System sends confirmation email
- FR47: Admin processes opt-out within 24 hours
- FR48: System removes opted-out DAOs from leaderboard
- FR49: Retain historical data internally (not public)
- FR58: Admin can view pending opt-out requests
- FR59: Admin can mark opt-outs as processed
- FR64: GDPR right to erasure compliance

**Epic 5: Report Ordering & Conversion Funnel**
- FR37: View "Get Report" CTA in expanded rows
- FR38: Click CTA to navigate to ordering flow
- FR39: See pricing information on CTA (€49-149)
- FR40: Distinguish Featured Analysis vs Customer Report badges
- FR41: Customer Reports show badge on leaderboard
- FR42: DAOs can opt-in to public listing after purchase
- FR67: Share leaderboard on social media
- FR68: Share specific DAO entries with pre-filled text
- FR69: Generate Open Graph tags for previews
- FR70: Generate Twitter Card tags
- FR71: Display "Get Report" CTA in expanded rows (pricing)
- FR72: Display "Share on X" button in expanded rows
- FR73: Track CTA click-through rates
- FR74: Track social share button clicks

**Epic 6: Weekly Automation & Historical Tracking**
- FR10: Store historical GVS snapshots (also in Epic 1)
- FR11: Calculate week-over-week deltas (also in Epic 1)
- FR12: Execute automated weekly GVS recalculation job
- FR17: Display change indicators (also in Epic 2)

**Epic 7: Admin Operations Dashboard**
- FR50: Admin can manually trigger GVS recalc for specific DAO
- FR51: Admin can manually trigger GVS recalc for all DAOs
- FR52: Admin can view job execution logs
- FR53: Admin can view failed job error messages
- FR54: Admin can add DAOs to Featured Analysis list
- FR55: Admin can remove DAOs from Featured Analysis list
- FR56: Admin can override confidence scores
- FR57: Admin can pause weekly job execution
- FR58: Admin can view pending opt-out requests (also in Epic 4)
- FR59: Admin can mark opt-outs as processed (also in Epic 4)

**Epic 8: Production Readiness & Monitoring**
- FR75: Leaderboard page LCP < 2.0s (p95)
- FR76: Leaderboard page FCP < 1.5s (p95)
- FR77: Implement ISR caching (1-hour revalidation)
- FR78: Gracefully handle Snapshot API failures
- FR79: Gracefully handle Ethereum RPC failures
- FR80: Retry failed API calls with exponential backoff
- FR81: Implement rate limiting (100 req/min per IP)
- FR82: Generate sitemap.xml
- FR83: Implement robots.txt
- FR84: Add canonical URLs
- FR85: Implement structured data (JSON-LD) for leaderboard
- FR86: Implement structured data for methodology page
- FR87: Track Core Web Vitals
- FR88: Track page views
- FR89: Track CTA conversion rates
- FR90: Track social share rates
- FR91: Track opt-out rate
- FR92: Track repeat visitor rate
- FR93: Send alerts when LCP increases >20%
- All 119 NFRs (Performance, Security, Scalability, Accessibility, Reliability, Maintainability, Integration, SEO)

**Epic 9: UX Enhancements (MVP+1 Week)**
- UX-P1-1: Mobile tap affordance (entire card tappable)
- UX-P1-2: First-time user onboarding (dismissible banner)
- UX-P1-3: Top 5 Control visual indicators (severity icons)
- UX-P2-1: Empty state for low DAO count (lead-gen CTA)
- UX-P2-2: Sticky "Last Updated" header on scroll
- UX-P2-3: Historical trend sparklines (4-week mini charts)

### Coverage Summary

- **Epic 0**: 14 requirements (ARCH-SETUP + ARCH-TECH)
- **Epic 1**: 11 requirements (FR1-FR11)
- **Epic 2**: 16 requirements (FR13-FR26, UX-P0-1, UX-P0-2)
- **Epic 3**: 17 requirements (FR27-FR36, FR60-FR66 partial)
- **Epic 4**: 10 requirements (FR43-FR49, FR58-FR59, FR64)
- **Epic 5**: 16 requirements (FR37-FR42, FR67-FR74)
- **Epic 6**: 4 requirements (FR10-FR12, FR17 - some overlap with Epic 1/2)
- **Epic 7**: 10 requirements (FR50-FR59)
- **Epic 8**: 138 requirements (FR75-FR93 + all 119 NFRs)
- **Epic 9**: 6 requirements (UX-P1-1 to UX-P2-3)

**Total: 242 requirement mappings** (some FRs mapped to multiple epics due to cross-cutting concerns)

## Epic List

### Epic 0: Project Foundation & Development Setup

**Goal:** Establish complete Next.js 14 development environment with all dependencies, configurations, and architectural patterns ready for feature implementation.

**User Outcome:** Development environment ready for all feature implementation (Foundation epic - enables all others)

**FRs Covered:** ARCH-SETUP-1 through ARCH-SETUP-6, ARCH-TECH-1 through ARCH-TECH-8 (14 requirements)

**Epic Description:**
Initialize the ChainSights project using Next.js 14 App Router with TypeScript strict mode, Tailwind CSS, and ESLint. Install all core dependencies including Drizzle ORM for database, wagmi/viem for Web3 integration, Stripe for payments, Anthropic SDK for AI analysis, and Vitest/Playwright for testing. Configure the database schema with snake_case naming convention, set up environment variables, establish Server Components pattern as default, and configure Vercel deployment. This epic provides the technical foundation that all feature epics depend on.

**Dependencies:** None (foundation epic)

**Enables:** All other epics (Epic 1-9)

---

### Epic 1: Governance Vitality Score (GVS) Calculation Engine

**Goal:** Build accurate, reliable governance scoring system that calculates GVS for any DAO using the weighted 4-component formula.

**User Outcome:** System can calculate trustworthy governance health scores that populate the leaderboard with confidence indicators.

**FRs Covered:** FR1-FR11 (11 requirements)

**Epic Description:**
Implement the core GVS calculation engine with all 4 components: Human Participation Rate (HPR - 35% weight), Delegate Engagement Index (DEI - 25% weight), Power Dynamics Index (PDI - 20% weight), and Grassroots Participation Index (GPI - 20% weight). Build data pipeline to fetch governance data from Snapshot.org API and token distribution from Ethereum blockchain. Implement confidence scoring based on data completeness (High/Medium/Low) and exclude DAOs with <50% confidence. Store historical GVS snapshots with timestamps in database and calculate week-over-week deltas. This is the analytical heart of the product - scores must be accurate before going public.

**Dependencies:** Epic 0 (Project Foundation)

**Enables:** Epic 2 (Leaderboard - provides data), Epic 3 (Methodology - explains scoring), Epic 6 (Weekly Automation - uses calculation), Epic 7 (Admin - manages calculation)

---

### Epic 2: Public Rankings Leaderboard (MVP)

**Goal:** Launch public-facing leaderboard where users can view, understand, and navigate DAO governance rankings with full accessibility compliance.

**User Outcome:** Users can view ranked DAOs, filter by category, sort by columns, expand for component details, share socially, and access seamlessly on any device.

**FRs Covered:** FR13-FR26, UX-P0-1, UX-P0-2 (16 requirements)

**Epic Description:**
Build the main leaderboard page at /rankings displaying ranked table with rank, DAO name, GVS score, score label (Vital/Stable/Concerning/Critical), category, and badge type. Implement color-coded score bars with texture patterns for color-blind accessibility (WCAG 2.1 Level AA). Add week-over-week change indicators with directional arrows (+/- deltas). Build filter by category (All/DeFi/Infrastructure/Public Goods) and sort by columns. Enable row expansion to show component breakdown (HPR, DEI, PDI, GPI). Display "last updated" timestamp. Ensure full keyboard navigation (tab through rows, enter to expand) and screen reader support (semantic HTML, ARIA labels). Implement responsive design for mobile (vertical scrolling) and desktop (full-width table). Fix CTA pricing ambiguity (confirm €49 pricing). This is the MVP centerpiece - the primary user interface.

**Dependencies:** Epic 0 (Project Foundation), Epic 1 (GVS Engine - provides data)

**Enables:** Epic 4 (Opt-Out - adds removal from this leaderboard), Epic 5 (CTAs - adds conversion elements), Epic 9 (UX Enhancements - improves this interface)

---

### Epic 3: Methodology Transparency & Legal Foundation

**Goal:** Build trust and legal protection through comprehensive methodology documentation and legal disclaimers.

**User Outcome:** Users can understand how GVS scores are calculated, trust the methodology, see legal disclaimers, and know their opt-out rights.

**FRs Covered:** FR27-FR36, FR60-FR66 (partial) (17 requirements)

**Epic Description:**
Create methodology page at /rankings/methodology explaining the GVS calculation approach with detailed breakdown of each component (HPR, DEI, PDI, GPI) and their weighting (35%, 25%, 20%, 20%). Document data sources (Snapshot.org, Ethereum RPC providers) and update frequency (weekly, Sunday midnight UTC). Display legal disclaimers ("opinions not facts," no investment advice), opt-out policy (24-hour removal guarantee), and confidence scoring methodology (High/Medium/Low criteria). Include contact information for questions/disputes, methodology version number, and last updated date. Implement input validation to prevent XSS and SQL injection attacks. Link privacy policy from footer. This epic is critical for legal protection before launch and builds credibility as "the methodology behind the rankings."

**Dependencies:** Epic 0 (Project Foundation), Epic 1 (GVS Engine - explains this)

**Enables:** No direct dependencies, but supports trust in Epic 2 (Leaderboard)

---

### Epic 4: Opt-Out Management System

**Goal:** Provide DAOs with guaranteed 24-hour removal from public rankings to avoid "hostage" perception and ensure GDPR compliance.

**User Outcome:** DAOs can submit opt-out request, receive confirmation, and see their DAO removed from leaderboard within 24 hours.

**FRs Covered:** FR43-FR49, FR58-FR59, FR64 (10 requirements)

**Epic Description:**
Build opt-out form at /rankings/opt-out with fields for DAO name, contact email, and optional reason. Implement form validation and submission workflow. Send confirmation email to requester upon submission. Create admin interface to view pending opt-out requests and mark them as processed. Implement system logic to remove opted-out DAOs from public leaderboard within 24 hours while retaining historical data internally (not public). Maintain audit log for all opt-out requests. Ensure GDPR right to erasure compliance. This epic is critical for legal compliance and preventing PR disasters - launches with MVP.

**Dependencies:** Epic 0 (Project Foundation), Epic 2 (Leaderboard - removes from this)

**Enables:** Legal safety for public launch

---

### Epic 5: Report Ordering & Conversion Funnel

**Goal:** Enable revenue generation through report CTAs and maximize organic reach through social sharing.

**User Outcome:** Users can purchase detailed DAO governance reports and share rankings on social media, driving traffic and conversions.

**FRs Covered:** FR37-FR42, FR67-FR74 (16 requirements)

**Epic Description:**
Add "Get Report" CTA in expanded DAO rows with clear pricing (€49-149 confirmed). Implement CTA navigation to report ordering flow (manual fulfillment OK for MVP). Display badge system distinguishing Featured Analysis (ChainSights-commissioned) vs Customer Report (DAO-purchased with opt-in). Build social share buttons for Twitter/X and LinkedIn with pre-filled text. Generate Open Graph tags and Twitter Card tags for rich social media previews. Implement tracking for CTA click-through rates and social share button clicks. This epic creates the conversion funnel and viral loop that drives revenue and growth.

**Dependencies:** Epic 0 (Project Foundation), Epic 2 (Leaderboard - adds CTAs to this)

**Enables:** Revenue generation, organic growth through social sharing

---

### Epic 6: Weekly Automation & Historical Tracking

**Goal:** Keep rankings fresh automatically with weekly updates, enabling "movement-based virality" through visible ranking changes.

**User Outcome:** Rankings update automatically every Sunday with change indicators showing movement over time.

**FRs Covered:** FR10-FR12, FR17 (4 requirements, some overlap with Epic 1/2)

**Epic Description:**
Implement automated weekly GVS recalculation job using Vercel Cron (Sunday midnight UTC). Job fetches latest data, recalculates GVS for all DAOs, stores new snapshots, and calculates deltas. Implement parallel processing (10 DAOs simultaneously) for efficiency. Add job execution logging (start time, duration, status, errors). Implement retry logic for failed jobs and Slack alert to admin on failure. Ensure historical snapshots are stored with timestamps for week-over-week comparison. Display change indicators on leaderboard (implemented in Epic 2, populated by this epic). This epic enables the "movement" that drives recurring engagement - launches Week 2 after static MVP.

**Dependencies:** Epic 0 (Project Foundation), Epic 1 (GVS Engine - runs this calculation)

**Enables:** Week-over-week changes in Epic 2 (Leaderboard), historical trend analysis

---

### Epic 7: Admin Operations Dashboard

**Goal:** Provide admin tools to manually manage rankings, monitor system health, and handle operations tasks.

**User Outcome:** Admin can manually trigger recalculations, view job logs, manage Featured Analysis list, and process opt-out requests.

**FRs Covered:** FR50-FR59 (10 requirements)

**Epic Description:**
Build admin dashboard at /admin with basic authentication (Vercel basic auth or session-based). Implement manual GVS recalculation triggers for specific DAOs or all DAOs. Display job execution logs with start time, duration, status, and error messages. Build interface to add/remove DAOs from Featured Analysis list and override confidence scores for specific DAOs. Add controls to pause/resume weekly job execution. Integrate opt-out request management (view pending requests, mark as processed). Basic MVP version acceptable - can be enhanced post-launch.

**Dependencies:** Epic 0 (Project Foundation), Epic 1 (GVS Engine - manages this), Epic 2 (Leaderboard - affects this), Epic 4 (Opt-Out - integrates with this)

**Enables:** Operational control and management

---

### Epic 8: Production Readiness & Monitoring

**Goal:** Ensure system meets all quality standards for public launch: performance, security, reliability, accessibility, SEO.

**User Outcome:** Users experience fast page loads (<2s LCP), reliable uptime (99.5%), secure connections, accessible interface (WCAG 2.1 AA), and discoverable content.

**FRs Covered:** FR75-FR93 + All 119 NFRs (138 requirements total)

**Epic Description:**
This cross-cutting epic ensures production quality across all features. Implement Core Web Vitals optimization (LCP <2s, FCP <1.5s, CLS <0.1). Configure ISR caching (1-hour revalidation), asset budgets (JS <150KB), and error recovery (serve cached data on API failure). Implement rate limiting (100 req/min per IP), HTTPS with HSTS, Content Security Policy, and GDPR compliance. Set up monitoring with UptimeRobot (ping /rankings every 5 minutes), Sentry error tracking, and Vercel Analytics. Configure SEO (sitemap.xml, robots.txt, structured data, Open Graph tags). Ensure 99.5% uptime target, database backups (Neon automatic daily), and deployment rollback capability (Vercel 1-click). Implement alerting for performance degradation (LCP increases >20%). This epic can be implemented incrementally throughout other epics with final validation before launch.

**Dependencies:** Epic 0 (Project Foundation), applies to all feature epics (Epic 1-7, 9)

**Enables:** Production launch readiness, operational excellence

---

### Epic 9: UX Enhancements (MVP+1 Week)

**Goal:** Improve first-time user comprehension, mobile interaction, and visual clarity based on UX validation findings.

**User Outcome:** First-time visitors understand leaderboard faster, mobile users tap entire cards naturally, severity indicators are instantly clear.

**FRs Covered:** UX-P1-1, UX-P1-2, UX-P1-3, UX-P2-1, UX-P2-2, UX-P2-3 (6 requirements)

**Epic Description:**
Implement P1 UX enhancements for MVP+1 Week: (1) Make entire mobile card tappable with chevron indicator (rotate on expand), (2) Add dismissible first-visit info banner with localStorage flag explaining leaderboard purpose, (3) Add visual severity indicators to Top 5 Control column (✓/⚠️/🔶/🔴 icons for quick scanning). Implement P2 nice-to-have enhancements post-MVP: (4) Empty state CTA when category filter has <5 DAOs (lead-gen opportunity), (5) Sticky "Last Updated" header appearing on scroll with update schedule dropdown, (6) Historical trend sparklines (4-week mini charts) next to change indicators. P1 items prioritized for Week 2, P2 items as growth features.

**Dependencies:** Epic 0 (Project Foundation), Epic 2 (Leaderboard - enhances this)

**Enables:** Improved user experience, lower bounce rate, higher engagement

---

### Epic Implementation Sequence

**MVP Launch (Week 1):**
- Epic 0: Project Foundation ✅
- Epic 1: GVS Calculation Engine ✅
- Epic 2: Public Rankings Leaderboard ✅
- Epic 3: Methodology & Legal ✅
- Epic 4: Opt-Out Management ✅
- Epic 5: Report CTAs & Social ✅
- Epic 8: Production Readiness (partial) ✅

**MVP+1 Week (Week 2):**
- Epic 6: Weekly Automation ✅
- Epic 9: UX Enhancements (P1 items) ✅

**Post-MVP:**
- Epic 7: Admin Dashboard (can be basic initially)
- Epic 8: Production Readiness (ongoing refinement)
- Epic 9: UX Enhancements (P2 items)

---

## Epic 0: Project Foundation & Development Setup

**Goal:** Establish complete Next.js 14 development environment with all dependencies, configurations, and architectural patterns ready for feature implementation.

**User Outcome:** Development environment ready for all feature implementation (Foundation epic - enables all others)

**Requirements Covered:** ARCH-SETUP-1 through ARCH-SETUP-6, ARCH-TECH-1 through ARCH-TECH-8 (14 requirements)

---

### Story 0.1: Initialize Next.js 14 Project with Core Configuration

As a developer,
I want to initialize a Next.js 14 project with TypeScript, Tailwind CSS, and ESLint configured,
So that the project foundation follows modern best practices and architectural patterns.

**Acceptance Criteria:**

**Given** an empty project directory
**When** I run the Next.js initialization command with specified flags
**Then** the project is created with App Router, TypeScript strict mode, Tailwind CSS, ESLint, src directory, and @/* import alias
**And** the project structure includes:
- `/src/app` directory for App Router pages
- `/src/components` directory for React components
- `/src/lib` directory for utilities and business logic
- `tsconfig.json` with `strict: true` and path aliases configured
- `tailwind.config.js` with custom theme colors (navy, aqua)
- `.eslintrc.json` with Next.js recommended rules
**And** I can run `npm run dev` and see the Next.js welcome page at localhost:3000
**And** TypeScript compilation succeeds with zero errors

**Implementation Notes:**
- Execute: `npx create-next-app@latest . --typescript --tailwind --eslint --app --src-dir --import-alias "@/*" --no-git --use-npm`
- Customize `tailwind.config.js` with ChainSights brand colors
- Verify `tsconfig.json` has `"strict": true` enabled

---

### Story 0.2: Install and Configure Database Layer (Drizzle ORM + Neon)

As a developer,
I want to install Drizzle ORM with Neon Postgres and configure the connection,
So that the application can persist data with type-safe database queries.

**Acceptance Criteria:**

**Given** the Next.js project is initialized
**When** I install drizzle-orm, drizzle-kit, and @neondatabase/serverless packages
**Then** all database dependencies are installed successfully
**And** `src/lib/db/index.ts` exports a configured Drizzle client
**And** `src/lib/db/schema.ts` exists with snake_case table naming convention
**And** `drizzle.config.ts` is configured with Neon connection string from environment variable
**And** environment variables include:
- `DATABASE_URL` in `.env.example` (with dummy value)
- `DATABASE_URL` in `.env.local` (for local development)
**And** I can run `npm run db:generate` to generate migrations
**And** I can run `npm run db:push` to push schema to database

**Implementation Notes:**
- Install: `npm install drizzle-orm @neondatabase/serverless`
- Install dev: `npm install -D drizzle-kit`
- Create `src/lib/db/index.ts` with Neon connection pooling
- Add npm scripts: `"db:generate": "drizzle-kit generate"`, `"db:push": "drizzle-kit push"`
- Set `DATABASE_URL` format: `postgresql://user:pass@host/db?sslmode=require`

---

### Story 0.3: Install and Configure External Service Dependencies

As a developer,
I want to install all external service SDKs and configure their initialization,
So that the application can integrate with Stripe, Anthropic, Resend, and QStash services.

**Acceptance Criteria:**

**Given** the project has core dependencies installed
**When** I install external service packages
**Then** the following packages are installed:
- `stripe` (payments)
- `@anthropic-ai/sdk` (AI analysis)
- `resend` (email)
- `@upstash/qstash` (job scheduling)
- `wagmi`, `viem` (Web3 - crypto payments future)
- `@walletconnect/ethereum-provider`, `@coinbase/wallet-sdk` (Web3 wallets)
**And** environment variables include placeholders in `.env.example`:
- `STRIPE_SECRET_KEY`
- `STRIPE_WEBHOOK_SECRET`
- `ANTHROPIC_API_KEY`
- `RESEND_API_KEY`
- `QSTASH_TOKEN`
- `NEXT_PUBLIC_SITE_URL`
**And** `src/lib/stripe.ts` exports configured Stripe client
**And** `src/lib/anthropic.ts` exports configured Anthropic client
**And** `src/lib/resend.ts` exports configured Resend client
**And** All SDK clients initialize successfully without errors

**Implementation Notes:**
- Install: `npm install stripe @anthropic-ai/sdk resend @upstash/qstash wagmi viem @walletconnect/ethereum-provider @coinbase/wallet-sdk`
- Create lib files that lazy-load SDKs only when environment variables are present
- Follow pattern: Check for env var → throw helpful error if missing in production

---

### Story 0.4: Set Up Testing Framework and Tools

As a developer,
I want to install and configure Vitest and Playwright testing frameworks,
So that the application has comprehensive unit, integration, and E2E testing capabilities.

**Acceptance Criteria:**

**Given** the project has core dependencies installed
**When** I install testing dependencies
**Then** the following packages are installed:
- `vitest`, `@vitest/ui` (unit/integration testing)
- `@playwright/test` (E2E testing)
- `@axe-core/playwright` (accessibility testing)
- `@testing-library/react`, `@testing-library/jest-dom` (React testing utilities)
**And** `vitest.config.ts` is configured with:
- Test environment: `jsdom`
- Coverage provider: `v8`
- Path aliases matching tsconfig.json (`@/*`)
**And** `playwright.config.ts` is configured with:
- Base URL: `http://localhost:3000`
- Test directory: `tests/e2e`
- Browsers: Chromium, Firefox, WebKit
**And** npm scripts include:
- `"test"`: Run Vitest in watch mode
- `"test:unit"`: Run unit tests once
- `"test:integration"`: Run integration tests
- `"test:coverage"`: Generate coverage report
- `"test:e2e"`: Run Playwright E2E tests
**And** Test directories exist: `tests/unit/`, `tests/integration/`, `tests/e2e/`
**And** I can run `npm test` and see Vitest watch mode
**And** I can run `npm run test:e2e` and Playwright launches successfully

**Implementation Notes:**
- Install: `npm install -D vitest @vitest/ui @playwright/test @axe-core/playwright @testing-library/react @testing-library/jest-dom`
- Run: `npx playwright install` to download browser binaries
- Create sample test file: `tests/unit/example.test.ts` to verify setup

---

### Story 0.5: Configure Vercel Deployment and Environment Variables

As a developer,
I want to configure Vercel deployment settings and environment variable management,
So that the application can be deployed to production with proper configuration.

**Acceptance Criteria:**

**Given** the project has all dependencies installed and configured
**When** I configure Vercel deployment
**Then** `.vercelignore` file exists excluding unnecessary files
**And** `vercel.json` exists with configuration for:
- Build command: `npm run build`
- Output directory: `.next`
- Environment variables grouped by: Production, Preview, Development
**And** `.env.example` documents all required environment variables with descriptions:
- Database (DATABASE_URL)
- Stripe (STRIPE_SECRET_KEY, STRIPE_WEBHOOK_SECRET)
- Anthropic (ANTHROPIC_API_KEY)
- Resend (RESEND_API_KEY)
- QStash (QSTASH_TOKEN)
- Public (NEXT_PUBLIC_SITE_URL)
**And** `README.md` includes setup instructions:
- Clone repository
- Copy `.env.example` to `.env.local`
- Fill in environment variables
- Run `npm install`
- Run `npm run db:push`
- Run `npm run dev`
**And** `package.json` includes all npm scripts for development workflow
**And** I can run `npm run build` and Next.js builds successfully
**And** Build output shows no TypeScript errors or ESLint warnings

**Implementation Notes:**
- Add to `.vercelignore`: `node_modules/`, `.env.local`, `tests/`, `.next/cache/`
- Ensure `vercel.json` if needed (optional for standard Next.js deployments)
- Document environment variable setup clearly in README
- Verify `npm run build` completes successfully before marking story done

---

**Epic 0 Complete:** 5 stories created covering all 14 ARCH requirements

---

## Epic 1: Governance Vitality Score (GVS) Calculation Engine

**Goal:** Build accurate, reliable governance scoring system that calculates GVS for any DAO using the weighted 4-component formula.

**User Outcome:** System can calculate trustworthy governance health scores that populate the leaderboard with confidence indicators.

**Requirements Covered:** FR1-FR11 (11 requirements)

---

### Story 1.1: Create Database Schema for GVS Data

As a developer,
I want to create database schema for storing DAO information and GVS snapshots,
So that the system can persist governance scores and historical data.

**Acceptance Criteria:**

**Given** the database connection is configured
**When** I define the schema in `src/lib/db/schema.ts`
**Then** the following tables are created with snake_case naming:
- **`daos` table:**
  - `id` (uuid, primary key)
  - `slug` (text, unique, not null) - URL-friendly identifier
  - `name` (text, not null) - Display name
  - `category` (enum: defi, infrastructure, public_goods)
  - `snapshot_space` (text, not null) - Snapshot.org space ID
  - `is_opted_out` (boolean, default false)
  - `created_at` (timestamp, default now)
  - `updated_at` (timestamp, default now)
- **`gvs_snapshots` table:**
  - `id` (uuid, primary key)
  - `dao_id` (uuid, foreign key to daos.id)
  - `gvs_score` (numeric, not null) - Overall score 0-10
  - `hpr_score` (numeric) - Human Participation Rate component
  - `dei_score` (numeric) - Delegate Engagement Index component
  - `pdi_score` (numeric) - Power Dynamics Index component
  - `gpi_score` (numeric) - Grassroots Participation Index component
  - `confidence_level` (enum: high, medium, low)
  - `data_completeness` (numeric) - Percentage 0-100
  - `snapshot_date` (timestamp, not null)
  - `created_at` (timestamp, default now)
**And** indexes are created on:
  - `daos.slug` (unique index)
  - `gvs_snapshots.dao_id` (index)
  - `gvs_snapshots.snapshot_date` (index for historical queries)
**And** I can run `npm run db:push` and tables are created in Neon database
**And** I can query both tables successfully using Drizzle ORM

**Implementation Notes:**
- Use Drizzle `pgTable`, `pgEnum` for type-safe schema
- Follow snake_case convention strictly (ARCH-TECH-1)
- Add inline comments for complex fields
- Create types exported from schema for use in application code

---

### Story 1.2: Implement Snapshot.org API Integration

As a developer,
I want to fetch governance proposal and voting data from Snapshot.org GraphQL API,
So that the system can access real DAO governance activity data for scoring.

**Acceptance Criteria:**

**Given** a DAO's Snapshot space ID
**When** I call the Snapshot API integration function
**Then** `src/lib/snapshot/client.ts` exports:
  - `fetchProposals(spaceId, options)` - Fetches proposals for a space
  - `fetchVotes(proposalId)` - Fetches votes for a specific proposal
  - `fetchSpace(spaceId)` - Fetches space metadata
**And** API client implements:
  - GraphQL query construction for Snapshot's API endpoint
  - Retry logic with exponential backoff (3 attempts) per NFR-INTEG-6
  - 30-second timeout per NFR-INTEG-7
  - Error handling with typed error responses
**And** API responses are typed with TypeScript interfaces in `src/lib/snapshot/types.ts`
**And** API client handles rate limiting (429 status) gracefully
**And** I can fetch proposals for "gitcoindao.eth" space and receive valid data
**And** Unit tests exist in `tests/unit/snapshot/client.test.ts` covering:
  - Successful API calls
  - Retry logic on transient failures
  - Timeout handling
  - Rate limit handling

**Implementation Notes:**
- Snapshot GraphQL endpoint: `https://hub.snapshot.org/graphql`
- Implement circuit breaker after 5 consecutive failures (NFR-INTEG-8)
- Cache responses for 15 minutes per architecture decision
- Handle Snapshot schema changes gracefully (NFR-INTEG-9)

---

### Story 1.3: Calculate Human Participation Rate (HPR) Component

As a developer,
I want to calculate the Human Participation Rate (HPR) component of GVS,
So that the score reflects unique human participation vs bot/whale voting patterns.

**Acceptance Criteria:**

**Given** proposal and voting data from Snapshot.org
**When** I call `calculateHPR(proposals, votes, options)` in `src/lib/gvs/hpr.ts`
**Then** the function returns an HPR score between 0-10
**And** HPR calculation logic:
  1. Identify unique voting wallets across all proposals
  2. Apply wallet clustering algorithm to detect Sybil/bot patterns
  3. Calculate percentage of "likely human" wallets vs total wallets
  4. Score on 0-10 scale based on human participation percentage
**And** Wallet clustering considers:
  - Voting timing patterns (same-block votes = suspicious)
  - Voting choice patterns (always votes same direction = suspicious)
  - Wallet age and transaction history
**And** Function accepts `lookbackPeriod` parameter (default: 90 days)
**And** Function returns metadata object including:
  - `totalVoters`: Total unique voting wallets
  - `likelyHumanVoters`: Count of likely human voters
  - `humanParticipationRate`: Percentage
  - `hprScore`: Final 0-10 score
**And** Unit tests cover:
  - High human participation (>70%) → score 8-10
  - Medium human participation (40-70%) → score 5-7
  - Low human participation (<40%) → score 0-4
  - Edge case: No votes → score 0
**And** Inline comments explain scoring thresholds and formulas

**Implementation Notes:**
- HPR weight in final GVS: 35% (highest weight)
- Algorithm should be documented in detail for methodology page
- Consider implementing simple heuristics first, optimize later

---

### Story 1.4: Calculate Delegate Engagement Index (DEI) Component

As a developer,
I want to calculate the Delegate Engagement Index (DEI) component of GVS,
So that the score reflects how actively delegates participate in governance.

**Acceptance Criteria:**

**Given** proposal and voting data with delegate information from Snapshot.org
**When** I call `calculateDEI(proposals, votes, delegates, options)` in `src/lib/gvs/dei.ts`
**Then** the function returns a DEI score between 0-10
**And** DEI calculation logic:
  1. Identify delegates (wallets with delegated voting power)
  2. Calculate delegate participation rate (voted proposals / total proposals)
  3. Weight by recency (recent participation weighted higher)
  4. Consider delegate turnover (new delegates = positive signal)
  5. Score on 0-10 scale
**And** Recency weighting formula:
  - Last 30 days: 1.0x weight
  - 30-60 days: 0.7x weight
  - 60-90 days: 0.4x weight
  - >90 days: 0.1x weight
**And** Function returns metadata object including:
  - `totalDelegates`: Count of delegates
  - `activeDelegates`: Count participating in last 90 days
  - `avgParticipationRate`: Average % of proposals voted on
  - `deiScore`: Final 0-10 score
**And** Unit tests cover:
  - High delegate engagement (>60% participation) → score 8-10
  - Medium engagement (30-60%) → score 5-7
  - Low engagement (<30%) → score 0-4
  - No delegates → score 5 (neutral, not penalized)
**And** Function handles missing delegate data gracefully

**Implementation Notes:**
- DEI weight in final GVS: 25%
- Some DAOs may not have formal delegation - handle gracefully
- Document formula in code comments for methodology page

---

### Story 1.5: Calculate Power Dynamics Index (PDI) Component

As a developer,
I want to calculate the Power Dynamics Index (PDI) component of GVS,
So that the score reflects voting power concentration and leadership turnover patterns.

**Acceptance Criteria:**

**Given** proposal and voting data with voting power information
**When** I call `calculatePDI(proposals, votes, options)` in `src/lib/gvs/pdi.ts`
**Then** the function returns a PDI score between 0-10
**And** PDI calculation logic combines three sub-metrics:
  1. **Gini coefficient trend** (measuring inequality in voting power distribution)
     - Calculate Gini for each month over 6-month period
     - Decreasing Gini = positive (power becoming more distributed)
     - Increasing Gini = negative (power concentrating)
  2. **Leadership turnover rate** (top 10 voters changing over time)
     - Compare top 10 voters month-over-month
     - Higher turnover = positive (new voices emerging)
  3. **Nakamoto coefficient** (minimum wallets controlling >50% voting power)
     - Higher Nakamoto = positive (more decentralized)
**And** Scoring weights for sub-metrics:
  - Gini trend: 40%
  - Leadership turnover: 30%
  - Nakamoto coefficient: 30%
**And** Function returns metadata object including:
  - `currentGini`: Current Gini coefficient (0-1)
  - `giniTrend`: Direction of change (increasing/decreasing/stable)
  - `nakamotoCoefficient`: Number of wallets to 50% power
  - `leadershipTurnoverRate`: Percentage month-over-month
  - `pdiScore`: Final 0-10 score
**And** Unit tests cover:
  - Improving power distribution → score 8-10
  - Stable power distribution → score 5-7
  - Worsening concentration → score 0-4
**And** Function handles insufficient historical data gracefully (returns neutral score)

**Implementation Notes:**
- PDI weight in final GVS: 20%
- Gini coefficient calculation requires sorting voting powers
- Nakamoto coefficient useful for "Top 5 Control" leaderboard metric

---

### Story 1.6: Calculate Grassroots Participation Index (GPI) Component

As a developer,
I want to calculate the Grassroots Participation Index (GPI) component of GVS,
So that the score reflects participation from small token holders and voting diversity.

**Acceptance Criteria:**

**Given** proposal and voting data with token holdings information
**When** I call `calculateGPI(proposals, votes, tokenHoldings, options)` in `src/lib/gvs/gpi.ts`
**Then** the function returns a GPI score between 0-10
**And** GPI calculation logic combines three sub-metrics:
  1. **Small holder participation** (voters with <1% of total supply)
     - Count small holders who voted
     - Higher participation = positive signal
  2. **Delegation breadth** (how widely delegated power is distributed)
     - Count unique delegates receiving delegations
     - More delegates = more distributed = positive
  3. **Vote diversity** (variety in voting choices, not unanimous)
     - Measure distribution of Yes/No/Abstain votes
     - More diverse = healthier debate = positive
**And** Scoring weights for sub-metrics:
  - Small holder participation: 50%
  - Delegation breadth: 30%
  - Vote diversity: 20%
**And** Function returns metadata object including:
  - `smallHolderCount`: Count of voters with <1% supply
  - `smallHolderParticipationRate`: Percentage
  - `uniqueDelegatesCount`: Number of unique delegates
  - `voteDiversityScore`: 0-1 score measuring variety
  - `gpiScore`: Final 0-10 score
**And** Unit tests cover:
  - High grassroots participation (>40% small holders) → score 8-10
  - Medium grassroots participation (20-40%) → score 5-7
  - Low grassroots participation (<20%) → score 0-4
**And** Function handles missing token holder data gracefully

**Implementation Notes:**
- GPI weight in final GVS: 25%
- "Small holder" threshold (1% supply) may need tuning per DAO
- Vote diversity prevents "rubber stamp" governance from scoring high

---

### Story 1.7: Aggregate GVS Score with Confidence Scoring

As a developer,
I want to aggregate the 4 component scores into final GVS with confidence indicator,
So that users see an overall governance health score with data quality transparency.

**Acceptance Criteria:**

**Given** calculated HPR, DEI, PDI, and GPI component scores
**When** I call `calculateGVS(components, dataCompleteness)` in `src/lib/gvs/aggregate.ts`
**Then** the function calculates final GVS using weighted formula:
  - `GVS = (HPR × 0.35) + (DEI × 0.25) + (PDI × 0.20) + (GPI × 0.25)`
**And** GVS score is normalized to 0-10 scale
**And** Confidence level is assigned based on data completeness:
  - **High confidence:** ≥80% data completeness (all components calculated successfully)
  - **Medium confidence:** 50-79% data completeness (some components missing or estimated)
  - **Low confidence:** <50% data completeness (significant data gaps)
**And** DAOs with <50% confidence are automatically excluded from public rankings (FR7)
**And** Function returns comprehensive GVS result object:
  - `gvsScore`: Final 0-10 score
  - `components`: Object with {hpr, dei, pdi, gpi} individual scores
  - `confidenceLevel`: "high" | "medium" | "low"
  - `dataCompleteness`: Percentage 0-100
  - `excludeFromRankings`: Boolean (true if confidence <50%)
  - `calculatedAt`: Timestamp
**And** Unit tests verify:
  - Weighted formula calculation accuracy
  - Confidence level assignment logic
  - Exclusion rule for low confidence (<50%)
  - All scores between 0-10 bounds
**And** Function logs warning if any component score is missing

**Implementation Notes:**
- This is the main entry point for GVS calculation
- Orchestrates all component calculators
- Component scores must mathematically sum correctly per NFR-REL-6

---

### Story 1.8: Store Historical Snapshots and Calculate Deltas

As a developer,
I want to store GVS snapshots in the database and calculate week-over-week deltas,
So that the leaderboard can display ranking movement and historical trends.

**Acceptance Criteria:**

**Given** a calculated GVS result for a DAO
**When** I call `saveGVSSnapshot(daoId, gvsResult)` in `src/lib/gvs/storage.ts`
**Then** the function inserts a new record into `gvs_snapshots` table with:
  - dao_id (foreign key)
  - gvs_score, hpr_score, dei_score, pdi_score, gpi_score
  - confidence_level, data_completeness
  - snapshot_date (current timestamp)
**And** the function returns the created snapshot record
**And** Zero data loss occurs (NFR-REL-4) - all snapshots persisted successfully
**And** Function `calculateDelta(daoId, currentSnapshot)` in same file:
  - Fetches previous snapshot for the DAO (7 days ago ±1 day tolerance)
  - Calculates delta: `currentScore - previousScore`
  - Returns delta object: `{delta: number, previousScore: number, direction: "up"|"down"|"stable"}`
**And** Delta calculation handles edge cases:
  - No previous snapshot → delta = null (new DAO)
  - Previous snapshot >10 days old → delta = null (stale comparison)
**And** Function `getLatestSnapshots(daoIds)` fetches most recent snapshot for multiple DAOs efficiently
**And** Unit tests cover:
  - Snapshot insertion success
  - Delta calculation with valid previous snapshot
  - Delta null cases (no previous, stale previous)
  - Bulk fetch performance (<100ms for 50 DAOs per NFR-PERF-9)
**And** Integration tests verify database round-trip with real Neon connection

**Implementation Notes:**
- Use Drizzle batch operations for bulk inserts (weekly job will need this)
- Index on (dao_id, snapshot_date) critical for delta queries
- Historical snapshots never deleted (zero data loss requirement)

---

**Epic 1 Complete:** 8 stories created covering all 11 GVS calculation requirements

---

## Epic 2: Public Rankings Leaderboard (MVP)

**Goal:** Launch public-facing leaderboard where users can view, understand, and navigate DAO governance rankings with full accessibility compliance.

**User Outcome:** Users can view ranked DAOs, filter by category, sort by columns, expand for component details, and access seamlessly on any device.

**Requirements Covered:** FR13-FR26, UX-P0-1, UX-P0-2 (16 requirements)

---

### Story 2.1: Create Leaderboard Server Component with Data Fetching

As a user,
I want to view a ranked list of DAOs sorted by governance score,
So that I can quickly see which DAOs have the healthiest governance.

**Acceptance Criteria:**

**Given** GVS snapshots exist in the database
**When** I navigate to `/rankings` page
**Then** a Server Component fetches the latest GVS snapshots for all non-opted-out DAOs
**And** data is fetched using `getLatestSnapshots()` from Epic 1
**And** DAOs with confidence level <50% are excluded (FR7)
**And** results are sorted by GVS score descending (highest first)
**And** page uses ISR caching with 1-hour revalidation (NFR-PERF-77)
**And** "Last updated" timestamp is displayed showing snapshot_date
**And** page load LCP <2.0s (p95) per NFR-PERF-2
**And** SEO meta tags include title, description, Open Graph tags (NFR-SEO-6)

**Implementation Notes:**
- File: `src/app/rankings/page.tsx`
- Use async Server Component pattern (no useEffect)
- Add `export const revalidate = 3600` for ISR
- Pass data to client components via props (ARCH-TECH-3)

---

### Story 2.2: Build Ranking Table UI Component

As a user,
I want to see each DAO's rank, name, score, and category in a clear table format,
So that I can scan the leaderboard quickly.

**Acceptance Criteria:**

**Given** leaderboard data is fetched
**When** the table renders
**Then** each row displays:
  - Rank number (1, 2, 3...)
  - DAO name (from daos table)
  - GVS score (0-10, displayed as "X.X/10")
  - Score label (Vital/Stable/Concerning/Critical) based on thresholds
  - Category badge (DeFi/Infrastructure/Public Goods)
  - Badge type (Featured Analysis vs Customer Report)
**And** table uses semantic HTML: `<table>`, `<thead>`, `<tbody>`, `<th scope="col">` (NFR-ACCESS-9)
**And** table has `<caption>` or `aria-label="DAO Rankings Table"` (NFR-ACCESS-10)
**And** desktop layout shows all columns in full-width table (FR24)
**And** mobile layout adapts to vertical scrolling (FR23)
**And** color contrast meets 4.5:1 for text, 7:1 for headings (NFR-ACCESS-1)

**Implementation Notes:**
- Component: `src/components/RankingsTable.tsx` (Server Component)
- Score thresholds: Vital 80-100, Stable 60-79, Concerning 40-59, Critical 0-39
- Use Tailwind responsive classes for mobile adaptation

---

### Story 2.3: Implement Score Labels with Accessibility Patterns

As a color-blind user,
I want score severity to be distinguishable without relying on color alone,
So that I can understand DAO health regardless of my color perception.

**Acceptance Criteria:**

**Given** a DAO with a GVS score
**When** the score bar is rendered
**Then** score bars use CSS gradient texture patterns in addition to color:
  - **Critical (0-39):** Red with diagonal stripes pattern
  - **Concerning (40-59):** Yellow with dotted pattern
  - **Stable (60-79):** Green with horizontal lines pattern
  - **Vital (80-100):** Green solid (no pattern)
**And** patterns are implemented using `repeating-linear-gradient` CSS
**And** score bars are distinguishable in grayscale mode
**And** axe-core accessibility audit passes with no color-only violations
**And** WCAG 2.1 Success Criterion 1.4.1 (Use of Color) is satisfied
**And** visual regression tests pass with patterns applied

**Implementation Notes:**
- Addresses UX-P0-1 (Critical accessibility fix)
- Component: `src/components/ScoreBar.tsx`
- Reference UX validation report for exact CSS patterns
- Test with grayscale browser extension

---

### Story 2.4: Add Filter and Sort Functionality

As a user,
I want to filter DAOs by category and sort by different columns,
So that I can find specific types of DAOs or view rankings by different criteria.

**Acceptance Criteria:**

**Given** the leaderboard is displayed
**When** I interact with filter and sort controls
**Then** category filter dropdown includes options:
  - All (default)
  - DeFi
  - Infrastructure
  - Public Goods
**And** clicking a category filters the table to show only DAOs in that category (FR18)
**And** sort controls allow sorting by:
  - Rank (default, ascending)
  - Score (descending)
  - Change (absolute value, descending)
  - Category (alphabetical)
**And** clicking a sort column toggles ascending/descending order
**And** `aria-sort` attribute updates to reflect current sort (NFR-ACCESS-10)
**And** screen reader announces "Currently sorted by score, descending" on sort action (NFR-ACCESS-11)
**And** filter/sort state persists in URL query params for shareability
**And** controls are client components with 'use client' directive

**Implementation Notes:**
- Component: `src/components/LeaderboardFilters.tsx` (Client Component)
- Use Next.js `useSearchParams` and `useRouter` for URL state
- Server Component reads query params and filters/sorts server-side for SEO

---

### Story 2.5: Build Expandable Row with Component Breakdown

As a user,
I want to expand a DAO row to see the detailed component score breakdown,
So that I can understand why a DAO received its overall score.

**Acceptance Criteria:**

**Given** a DAO row in the leaderboard
**When** I click the row or press Enter while focused
**Then** the row expands to show component breakdown section
**And** component breakdown displays:
  - HPR score with label "Human Participation Rate: X.X/10"
  - DEI score with label "Delegate Engagement Index: X.X/10"
  - PDI score with label "Power Dynamics Index: X.X/10"
  - GPI score with label "Grassroots Participation Index: X.X/10"
  - Brief explanation text for each component
**And** expanded section includes "Get Report" CTA button
**And** expanded section includes "Share on X" button
**And** clicking row again or pressing Enter collapses the section
**And** expand/collapse animation is smooth (CSS transition)
**And** `aria-expanded` attribute reflects current state (NFR-ACCESS-10)
**And** keyboard navigation works: Tab to row, Enter to toggle (FR25, NFR-ACCESS-5)
**And** screen reader announces state change (NFR-ACCESS-11)

**Implementation Notes:**
- Component: `src/components/ExpandableDAORow.tsx` (Client Component)
- Use React state for expand/collapse
- Animate with Tailwind transition classes
- Component scores from gvs_snapshots table

---

### Story 2.6: Display Week-Over-Week Change Indicators

As a user,
I want to see how each DAO's score changed from last week,
So that I can track governance improvement or decline over time.

**Acceptance Criteria:**

**Given** a DAO has a previous GVS snapshot from 7 days ago
**When** the current snapshot is displayed
**Then** change indicator shows:
  - Delta value (e.g., "+0.3", "-0.5", "—")
  - Directional arrow (↗ up, ↘ down, → no change)
  - Color coding (green positive, red negative, gray neutral)
**And** change indicators use both color AND arrows (NFR-ACCESS-4)
**And** `aria-label` describes change: "Increased by 0.3 points" (NFR-ACCESS-10)
**And** if no previous snapshot exists, display "New" badge
**And** if previous snapshot is stale (>10 days), display "—" (no comparison)
**And** deltas are calculated using `calculateDelta()` from Epic 1
**And** change indicators are visible in both desktop and mobile layouts

**Implementation Notes:**
- Component: `src/components/ChangeIndicator.tsx`
- Uses delta from gvs_snapshots query
- Threshold for "no change": ±0.1 points

---

### Story 2.7: Implement Mobile Responsive Layout

As a mobile user,
I want to view the leaderboard on my phone with optimized layout,
So that I can easily browse rankings on any device.

**Acceptance Criteria:**

**Given** I access `/rankings` on a mobile device (<768px width)
**When** the page renders
**Then** the table adapts to vertical scrolling layout (FR23)
**And** columns stack or hide appropriately:
  - Always visible: Rank, DAO name, Score
  - Hidden on mobile: Category (shown in expanded state)
  - Change indicator shown as compact badge
**And** touch targets are minimum 44×44px (NFR-ACCESS-13)
**And** row expansion works on tap (touch event)
**And** no horizontal scrolling required (NFR-ACCESS-reflow)
**And** text remains readable at 200% zoom (NFR-ACCESS-3)
**And** page works in both portrait and landscape orientation (NFR-ACCESS-14)
**And** mobile layout tested on iOS Safari and Android Chrome

**Implementation Notes:**
- Use Tailwind responsive breakpoints (sm:, md:, lg:)
- Test on real devices or browser DevTools
- Mobile-first approach in CSS

---

### Story 2.8: Add Keyboard Navigation and Screen Reader Support

As a keyboard-only or screen reader user,
I want to navigate the leaderboard without a mouse,
So that I can access all functionality regardless of my input method.

**Acceptance Criteria:**

**Given** I navigate using only keyboard
**When** I interact with the leaderboard
**Then** I can Tab through all interactive elements in logical order (NFR-ACCESS-5)
**And** visible focus indicator (2px aqua outline) shows on focused elements (NFR-ACCESS-2)
**And** pressing Enter on a DAO row expands/collapses it (FR25)
**And** pressing Enter on filter dropdown opens options
**And** arrow keys navigate within dropdown menus
**And** pressing Escape closes expanded rows or dropdowns
**And** "Skip to main content" link available at page top (NFR-ACCESS-6)
**And** no keyboard traps exist (NFR-ACCESS-8)
**And** screen reader announces:
  - Table structure with proper headers
  - Current sort order
  - Expanded/collapsed state changes
  - Dynamic content updates (filter changes)
**And** ARIA live regions announce filter/sort changes (NFR-ACCESS-11)
**And** manual testing with NVDA (Windows) and VoiceOver (macOS) passes

**Implementation Notes:**
- Use semantic HTML for automatic screen reader support
- Add explicit ARIA labels where needed
- Test focus order matches visual order
- Document keyboard shortcuts in accessibility statement

---

### Story 2.9: Fix CTA Pricing Ambiguity

As a user considering purchasing a report,
I want to see accurate pricing on the CTA button,
So that I'm not confused when I reach the checkout page.

**Acceptance Criteria:**

**Given** the expanded DAO row displays the "Get Report" CTA
**When** the CTA renders
**Then** the button text accurately reflects the actual pricing structure:
  - **Option A (Single Tier):** "Get Full Report — €49 →" if only one price exists
  - **Option B (Tiered Pricing):** "See Report Options → from €49" if multiple tiers exist
  - **Option C (Promotional):** "Get Report — €49 Launch Special →" if promotional pricing
**And** CTA price matches the price shown on the checkout/order page
**And** no user confusion in user testing about pricing
**And** conversion tracking shows no price-mismatch bounces
**And** pricing configuration is stored in environment variable or database for easy updates

**Implementation Notes:**
- Addresses UX-P0-2 (Critical pricing fix)
- Pricing decision finalized: €49 Deep Dive, €149 Governance Audit (see project_context.md)
- €99 legacy tier deprecated — NEVER display to users
- Component: Update `src/components/ExpandableDAORow.tsx` CTA section

---

### Story 2.10: Add "Last Updated" Timestamp Display

As a user,
I want to know when the rankings were last updated,
So that I can assess the freshness of the data.

**Acceptance Criteria:**

**Given** GVS snapshots have snapshot_date timestamps
**When** the leaderboard page renders
**Then** a timestamp display shows near the page header:
  - "Updated: December 19, 2024 at 12:05 AM UTC"
  - OR "Last updated: 2 hours ago" (relative time)
**And** timestamp uses the most recent snapshot_date from the dataset
**And** if data is >7 days old, warning banner shows: "Rankings may be outdated - last updated [date]"
**And** timestamp is visible on both desktop and mobile
**And** timestamp updates after ISR revalidation (1-hour cache)

**Implementation Notes:**
- Use date-fns or similar for formatting
- Component: `src/components/LastUpdatedBadge.tsx`
- Warning banner only shows if data staleness detected

---

**Epic 2 Complete:** 10 stories created covering all 16 leaderboard requirements
---

## Epic 3: Methodology Transparency & Legal Foundation

**Goal:** Build trust and legal protection through comprehensive methodology documentation and legal disclaimers.

**User Outcome:** Users can understand how GVS scores are calculated, trust the methodology, see legal disclaimers, and know their opt-out rights.

**Requirements Covered:** FR27-FR36, FR60-FR66 (partial) (17 requirements)

---

### Story 3.1: Create Methodology Page with GVS Explanation

As a user,
I want to understand how GVS scores are calculated,
So that I can trust the rankings and know what the scores mean.

**Acceptance Criteria:**

**Given** I navigate to `/rankings/methodology`
**When** the page loads
**Then** the page displays:
  - Overview of GVS (Governance Vitality Score) purpose and approach
  - Detailed explanation of each component: HPR (35% weight), DEI (25%), PDI (20%), GPI (25%)
  - For each component: Definition, calculation method, scoring thresholds
  - Data sources list: Snapshot.org, Ethereum blockchain providers
  - Update frequency: Weekly recalculation, Sunday midnight UTC
  - Confidence scoring methodology: High (≥80%), Medium (50-79%), Low (<50%)
**And** page uses Server Component (no client-side JavaScript for content)
**And** content is crawlable by search engines (no noindex)
**And** SEO meta tags include methodology-specific title/description
**And** internal link from leaderboard "Learn more about our methodology"
**And** methodology version number displayed (e.g., "v1.0 - Last updated: Dec 2024")

**Implementation Notes:**
- File: `src/app/rankings/methodology/page.tsx`
- Content can be markdown-based or hardcoded JSX
- Include mathematical formulas with clear explanations
- Link back to leaderboard page

---

### Story 3.2: Add Legal Disclaimers and Opt-Out Policy

As a DAO representative or legal-conscious user,
I want to see legal disclaimers and understand opt-out rights,
So that I know the limitations of the scores and how to request removal.

**Acceptance Criteria:**

**Given** I view the methodology page
**When** I scroll to the legal section
**Then** the page includes:
  - **Legal disclaimer**: "GVS scores represent ChainSights' opinion based on publicly available data, not statements of fact or investment advice"
  - **No endorsement**: "Rankings do not constitute endorsement or recommendation"
  - **Opt-out policy**: "DAOs may request removal from public rankings within 24 hours by submitting the opt-out form"
  - **Data accuracy**: "Scores calculated using best efforts; no guarantee of accuracy or completeness"
  - **GDPR compliance**: "We collect minimal data; see Privacy Policy for details"
  - **Contact information**: Email for questions/disputes (hello@chainsights.one)
**And** link to opt-out form: `/rankings/opt-out`
**And** link to privacy policy in footer
**And** disclaimers use plain language (avoid legalese where possible)
**And** section uses `<aside>` or similar semantic HTML

**Implementation Notes:**
- Consult legal advisor for final disclaimer text
- Store disclaimers in constants file for reuse across pages
- Ensure opt-out link is prominent (not buried)

---

### Story 3.3: Implement Input Validation for Security

As a developer,
I want all user inputs validated and sanitized,
So that the application is protected from XSS and SQL injection attacks.

**Acceptance Criteria:**

**Given** any form that accepts user input (opt-out form, future forms)
**When** data is submitted
**Then** input validation occurs using Zod schemas:
  - Email addresses validated with email regex
  - Text fields sanitized to remove HTML tags and script injection attempts
  - Maximum length limits enforced (e.g., 500 chars for reason field)
  - Required fields validated before processing
**And** validation happens server-side (never trust client-side validation alone)
**And** Zod schemas defined in `src/lib/validation/` directory
**And** Database queries use Drizzle ORM parameterized queries (never string concatenation)
**And** Error messages don't expose sensitive information or query structure
**And** Content Security Policy headers prevent inline script execution

**Implementation Notes:**
- Install zod if not already: `npm install zod`
- Example schema: `src/lib/validation/opt-out.ts`
- XSS prevention: Never use `dangerouslySetInnerHTML`
- SQL injection prevention: Drizzle ORM handles this automatically

---

### Story 3.4: Add Privacy Policy and Footer Links

As a user,
I want to access the privacy policy and legal pages,
So that I understand how my data is used.

**Acceptance Criteria:**

**Given** any page on the site
**When** I view the footer
**Then** footer includes links to:
  - Privacy Policy (`/privacy`)
  - Methodology (`/rankings/methodology`)
  - Opt-Out Form (`/rankings/opt-out`)
  - Contact (hello@chainsights.one)
**And** Privacy Policy page (`src/app/privacy/page.tsx`) includes:
  - What data we collect (minimal: opted-in emails only)
  - How data is used (report delivery, opt-out processing)
  - Data retention (90 days after report delivery unless marketing opt-in)
  - Cookie usage (Vercel Analytics only)
  - GDPR rights (right to erasure via opt-out)
  - Contact for data requests
**And** all legal pages use consistent layout and styling
**And** footer is visible on all pages (layout.tsx)

**Implementation Notes:**
- Privacy policy can be simplified markdown initially
- Update annually or when data practices change
- Ensure GDPR compliance (EU users)

---

**Epic 3 Complete:** 4 stories created covering all 17 methodology and legal requirements

---

## Epic 4: Opt-Out Management System

**Goal:** Provide DAOs with guaranteed 24-hour removal from public rankings to avoid "hostage" perception and ensure GDPR compliance.

**User Outcome:** DAOs can submit opt-out request, receive confirmation, and see their DAO removed from leaderboard within 24 hours.

**Requirements Covered:** FR43-FR49, FR58-FR59, FR64 (10 requirements)

---

### Story 4.1: Create Opt-Out Form Page

As a DAO representative,
I want to submit a request to remove my DAO from the leaderboard,
So that we can control our public governance data visibility.

**Acceptance Criteria:**

**Given** I navigate to `/rankings/opt-out`
**When** the page loads
**Then** a form displays with fields:
  - DAO Name (text input, required)
  - Contact Email (email input, required)
  - Reason for opt-out (textarea, optional, 500 char max)
**And** form includes text explaining:
  - Removal guaranteed within 24 hours
  - Historical data retained internally but not publicly displayed
  - Future recalculations will skip this DAO
  - Can request re-inclusion by emailing hello@chainsights.one
**And** form validates inputs using Zod schema from Epic 3
**And** submit button triggers client-side validation first
**And** form uses accessible labels, ARIA attributes, error messaging
**And** form styled consistently with site design

**Implementation Notes:**
- File: `src/app/rankings/opt-out/page.tsx`
- Form component: `src/components/OptOutForm.tsx` (Client Component)
- Validation schema: `src/lib/validation/opt-out.ts`

---

### Story 4.2: Implement Opt-Out Submission and Email Confirmation

As a DAO representative submitting an opt-out request,
I want to receive immediate confirmation that my request was received,
So that I know the process has started.

**Acceptance Criteria:**

**Given** I submit the opt-out form with valid data
**When** the form is processed
**Then** data is sent to API route `/api/opt-out/route.ts`
**And** API route:
  - Validates input using Zod schema
  - Creates record in `opt_out_requests` table (new table):
    - `id` (uuid, primary key)
    - `dao_name` (text, not null)
    - `contact_email` (text, not null)
    - `reason` (text, nullable)
    - `status` (enum: pending, processed)
    - `requested_at` (timestamp, default now)
    - `processed_at` (timestamp, nullable)
  - Sends confirmation email via Resend:
    - Subject: "DAO Opt-Out Request Received - ChainSights"
    - Body: Confirms receipt, 24-hour removal guarantee, request ID
  - Returns success response with request ID
**And** user sees success message: "Request received! Check your email for confirmation. Removal within 24 hours."
**And** if email fails, request is still stored but user sees warning
**And** audit log records the opt-out request (FR66)

**Implementation Notes:**
- Create `opt_out_requests` table schema in Epic 1's schema file
- Email template: `src/lib/email/templates/opt-out-confirmation.tsx`
- Use Resend SDK from Epic 0 setup

---

### Story 4.3: Implement Opt-Out Processing Logic

As an admin (Mario),
I want to process opt-out requests and remove DAOs from public rankings,
So that we honor the 24-hour removal guarantee.

**Acceptance Criteria:**

**Given** a pending opt-out request exists
**When** admin marks it as processed
**Then** the system:
  - Updates `daos` table: Sets `is_opted_out = true` for the specified DAO
  - Updates `opt_out_requests` table: Sets `status = 'processed'`, `processed_at = now()`
  - Next leaderboard data fetch automatically excludes opted-out DAOs (query filter)
**And** opted-out DAOs do not appear in `/rankings` page
**And** historical `gvs_snapshots` data retained internally (not deleted)
**And** future weekly GVS calculation jobs skip opted-out DAOs
**And** admin can view pending requests in admin dashboard (Epic 7)
**And** GDPR right to erasure is satisfied within 24 hours (NFR-SEC-10)

**Implementation Notes:**
- Opt-out logic: `src/lib/dao/opt-out.ts`
- Admin processing integrated in Epic 7 (Admin Dashboard)
- Leaderboard query adds `WHERE is_opted_out = false`

---

**Epic 4 Complete:** 3 stories created covering all 10 opt-out requirements

---

## Epic 5: Report Ordering & Conversion Funnel

**Goal:** Enable revenue generation through report CTAs and maximize organic reach through social sharing.

**User Outcome:** Users can purchase detailed DAO governance reports and share rankings on social media, driving traffic and conversions.

**Requirements Covered:** FR37-FR42, FR67-FR74 (16 requirements)

---

### Story 5.1: Add Social Share Buttons with Pre-Filled Text

As a user,
I want to share interesting DAO rankings on social media,
So that I can discuss governance with my network and drive traffic to ChainSights.

**Acceptance Criteria:**

**Given** an expanded DAO row in the leaderboard
**When** the "Share on X" button is displayed
**Then** clicking the button:
  - Opens Twitter/X share dialog in new window
  - Pre-fills tweet text: "Check out [DAO Name]'s Governance Vitality Score: X.X/10 on @ChainSights"
  - Includes URL to leaderboard page (or future DAO-specific page)
  - Uses Twitter Web Intent API: `https://twitter.com/intent/tweet?text=...&url=...`
**And** LinkedIn share button (optional for MVP):
  - Opens LinkedIn share dialog
  - Pre-fills with similar text
**And** share buttons track clicks using analytics event (FR74)
**And** buttons use accessible labels and ARIA attributes
**And** buttons styled consistently with site design (icon + text or icon only on mobile)

**Implementation Notes:**
- Component: `src/components/ShareButtons.tsx`
- Twitter intent docs: https://developer.twitter.com/en/docs/twitter-for-websites/tweet-button/overview
- Track events: `trackEvent('social_share', {platform: 'twitter', dao: daoName})`

---

### Story 5.2: Implement Open Graph and Twitter Card Tags

As a user sharing the leaderboard link,
I want rich previews to display on social media,
So that the shared content looks professional and attractive.

**Acceptance Criteria:**

**Given** the `/rankings` page or future `/rankings/[slug]` pages
**When** social media platforms crawl the URL
**Then** meta tags include:
  - **Open Graph:**
    - `og:title`: "DAO Governance Rankings | ChainSights"
    - `og:description`: "Compare DAO governance health with the Governance Vitality Score"
    - `og:image`: Preview image (leaderboard screenshot or branded graphic)
    - `og:url`: Canonical URL
    - `og:type`: "website"
  - **Twitter Card:**
    - `twitter:card`: "summary_large_image"
    - `twitter:title`, `twitter:description`, `twitter:image` (same as OG)
    - `twitter:site`: "@ChainSights"
**And** preview image is optimized (1200x630px, <500KB)
**And** images stored in `/public/og/` directory
**And** meta tags defined in page metadata export (Next.js 14 pattern)
**And** tested with Twitter Card Validator and Facebook Sharing Debugger

**Implementation Notes:**
- Add to `src/app/rankings/page.tsx` metadata export
- Generate OG image with Vercel OG Image or static design
- Update for dynamic DAO pages in future

---

### Story 5.3: Track Conversion Events with Analytics

As a product manager (Mario),
I want to track CTA clicks and social shares,
So that I can measure conversion funnel effectiveness.

**Acceptance Criteria:**

**Given** Vercel Analytics is configured (Epic 0)
**When** users interact with conversion points
**Then** the following events are tracked:
  - `cta_click`: When "Get Report" button clicked (FR73)
  - `social_share`: When share button clicked (FR74)
  - `methodology_view`: When methodology page viewed
  - `row_expand`: When DAO row expanded (engagement metric)
**And** events include metadata:
  - `dao_name` or `dao_id`
  - `score` (GVS score)
  - `platform` (for social shares: twitter, linkedin)
**And** tracking function: `trackEvent(eventName, properties)`
**And** events visible in Vercel Analytics dashboard
**And** no PII (personally identifiable information) tracked
**And** tracking respects user privacy (no cross-site tracking)

**Implementation Notes:**
- Use Vercel Analytics `track()` function
- Util file: `src/lib/analytics.ts`
- Track server-side page views automatically (Vercel Analytics default)
- Client-side events tracked on CTA/button clicks

---

###Story 5.4: Implement Report Ordering CTA (Manual Fulfillment for MVP)

As a user interested in a detailed governance report,
I want to click the CTA and be guided to order a report,
So that I can get in-depth analysis for a specific DAO.

**Acceptance Criteria:**

**Given** the expanded DAO row displays "Get Report" CTA
**When** I click the CTA
**Then** for MVP (manual fulfillment):
  - User is directed to a simple order form page `/order/[dao-slug]`
  - Form collects: Name, Email, DAO Name (pre-filled), Optional message
  - Form submission sends email to Mario (hello@chainsights.one) with order details
  - User sees confirmation: "Order received! We'll contact you within 24 hours with payment details and delivery timeline."
**And** CTA button shows correct pricing per Story 2.9 (UX-P0-2 fix)
**And** CTA click tracked with analytics event (FR73)
**And** badge system distinguishes "Featured Analysis" vs "Customer Report" on leaderboard (FR40-FR41)
**And** manual Stripe invoice sent via email for payment (acceptable for MVP)
**And** after payment, DAO can opt-in to public "Customer Report" badge

**Implementation Notes:**
- MVP approach: Manual fulfillment acceptable (save 10-15 hours automation)
- Future (Phase 2): Integrate Stripe checkout, automated report generation
- Order form: `src/app/order/[dao-slug]/page.tsx`
- Email notification to Mario for order processing

---

**Epic 5 Complete:** 4 stories created covering all 16 conversion requirements

---

## Epic 6: Weekly Automation & Historical Tracking

**Goal:** Keep rankings fresh automatically with weekly updates, enabling "movement-based virality" through visible ranking changes.

**User Outcome:** Rankings update automatically every Sunday with change indicators showing movement over time.

**Requirements Covered:** FR10-FR12, FR17 (4 requirements, some overlap with Epic 1)

---

### Story 6.1: Implement Weekly GVS Recalculation Cron Job

As the system,
I want to automatically recalculate GVS scores every Sunday,
So that rankings stay fresh without manual intervention.

**Acceptance Criteria:**

**Given** Vercel Cron is configured
**When** Sunday midnight UTC arrives
**Then** a cron job executes via API route `/api/jobs/recalculate-gvs/route.ts`
**And** job fetches all non-opted-out DAOs from `daos` table
**And** job processes DAOs in parallel (10 simultaneously) using `Promise.allSettled()`
**And** for each DAO:
  - Fetches latest governance data from Snapshot.org (Epic 1)
  - Calculates HPR, DEI, PDI, GPI components (Epic 1)
  - Aggregates final GVS with confidence scoring (Epic 1)
  - Saves new snapshot to `gvs_snapshots` table (Epic 1)
  - Calculates delta from previous snapshot (Epic 1)
**And** job completes in <2 hours for 50 DAOs (NFR-PERF-7)
**And** job logs execution: start time, duration, DAOs processed, errors
**And** if job fails, retry once automatically, then alert Mario via Slack/email
**And** cron configuration in `vercel.json` or `package.json`:
  - Schedule: `0 0 * * 0` (Sunday midnight UTC)

**Implementation Notes:**
- Vercel Cron docs: https://vercel.com/docs/cron-jobs
- Use QStash for reliable cron (from Epic 0 dependencies)
- Job timeout: 10 minutes max per Vercel serverless limits
- Consider breaking into multiple smaller jobs if needed

---

### Story 6.2: Add Job Monitoring and Failure Alerts

As an admin (Mario),
I want to be notified if the weekly job fails,
So that I can intervene and ensure rankings stay updated.

**Acceptance Criteria:**

**Given** the weekly GVS recalculation job runs
**When** the job completes or fails
**Then** job execution is logged to database in `job_logs` table (new table):
  - `id` (uuid, primary key)
  - `job_name` (text): "weekly-gvs-recalculation"
  - `status` (enum: running, success, failed)
  - `started_at` (timestamp)
  - `completed_at` (timestamp, nullable)
  - `daos_processed` (integer)
  - `errors` (json, nullable): Array of error objects
**And** on success:
  - Log entry marked as success
  - No notification sent (silent success)
**And** on failure (after retry):
  - Log entry marked as failed with error details
  - Slack notification sent to Mario (optional: use Slack webhook)
  - Email notification sent to hello@chainsights.one
  - Alert includes: Error message, DAOs that failed, timestamp
**And** admin dashboard (Epic 7) displays recent job logs
**And** job monitoring satisfies NFR-REL-13

**Implementation Notes:**
- Create `job_logs` table schema
- Slack webhook setup: https://api.slack.com/messaging/webhooks
- Email via Resend SDK (already configured)
- Consider using Sentry for error tracking

---

**Epic 6 Complete:** 2 stories created covering all 4 automation requirements

---

## Epic 7: Admin Operations Dashboard

**Goal:** Provide admin tools to manually manage rankings, monitor system health, and handle operations tasks.

**User Outcome:** Admin can manually trigger recalculations, view job logs, manage Featured Analysis list, and process opt-out requests.

**Requirements Covered:** FR50-FR59 (10 requirements)

---

### Story 7.1: Create Admin Dashboard with Authentication

As an admin (Mario),
I want to access a protected admin dashboard,
So that I can manage the rankings system.

**Acceptance Criteria:**

**Given** I navigate to `/admin`
**When** the page loads
**Then** authentication is required using Vercel basic auth or simple password protection
**And** after authentication, dashboard displays:
  - System status overview (last job run, upcoming job)
  - Quick actions: Manual GVS recalculation, View logs, Process opt-outs
  - Recent activity feed (last 10 job logs, opt-out requests)
**And** dashboard is server-rendered for security (no client-side auth bypass)
**And** authentication method: Environment variable password check or Vercel Edge Middleware
**And** failed login attempts logged
**And** dashboard styled with minimal but functional UI (Tailwind)

**Implementation Notes:**
- File: `src/app/admin/page.tsx`
- Auth: Check header or session, redirect if unauthorized
- Middleware: `src/middleware.ts` protects `/admin/*` routes
- Consider NextAuth.js for future if more admins needed

---

### Story 7.2: Implement Manual GVS Recalculation Triggers

As an admin (Mario),
I want to manually trigger GVS recalculation for specific DAOs or all DAOs,
So that I can update scores on-demand outside the weekly schedule.

**Acceptance Criteria:**

**Given** I'm on the admin dashboard
**When** I use the manual recalculation controls
**Then** two options are available:
  - **Recalculate Single DAO:** Dropdown to select DAO, button to trigger
  - **Recalculate All DAOs:** Button to trigger full recalculation
**And** clicking trigger:
  - Sends request to `/api/admin/recalculate/route.ts`
  - API validates admin authentication
  - Executes same logic as weekly job (Epic 6) synchronously or async
  - Returns status: "Recalculation started for [DAO Name]" or "Recalculation started for all DAOs"
**And** progress indicator shows while processing (loading spinner)
**And** success/error message displayed after completion
**And** manual recalculations logged to `job_logs` table
**And** admin can view results immediately on leaderboard (ISR cache bypassed or short-lived)

**Implementation Notes:**
- API route: `src/app/api/admin/recalculate/route.ts`
- Use same GVS calculation functions from Epic 1
- For "all DAOs", consider async queue to avoid timeout

---

### Story 7.3: Display Job Execution Logs

As an admin (Mario),
I want to view historical job execution logs,
So that I can diagnose issues and verify the system is running correctly.

**Acceptance Criteria:**

**Given** I navigate to `/admin/logs`
**When** the page loads
**Then** a table displays recent job logs from `job_logs` table:
  - Columns: Job Name, Status, Started At, Completed At, Duration, DAOs Processed, Errors (if any)
  - Most recent logs first (sorted by started_at DESC)
  - Pagination or "Load More" for older logs
**And** clicking a log row expands to show:
  - Full error details (if failed)
  - List of DAOs processed
  - Any warnings or partial failures
**And** filter options: Status (all/success/failed), Date range
**And** export logs as CSV for offline analysis (optional nice-to-have)

**Implementation Notes:**
- Server Component fetches logs from database
- Consider limiting to last 100 logs for performance
- Failed jobs highlighted in red for visibility

---

### Story 7.4: Integrate Opt-Out Request Management

As an admin (Mario),
I want to view pending opt-out requests and mark them as processed,
So that I can honor the 24-hour removal guarantee.

**Acceptance Criteria:**

**Given** I navigate to `/admin/opt-outs`
**When** the page loads
**Then** a table displays pending opt-out requests from `opt_out_requests` table:
  - Columns: DAO Name, Contact Email, Reason, Requested At, Status
  - Only pending requests shown by default
  - Filter: Show all (pending + processed)
**And** each pending request has "Mark as Processed" button
**And** clicking button:
  - Calls `/api/admin/opt-out/process/route.ts`
  - Updates DAO's `is_opted_out = true` in `daos` table
  - Updates request `status = 'processed'`, `processed_at = now()`
  - Shows success message: "DAO removed from public rankings"
**And** processed requests show checkmark and timestamp
**And** admin can search/filter by DAO name or email
**And** opt-out processing time tracked (should be <24 hours from request)

**Implementation Notes:**
- Integrates with Epic 4 (Opt-Out) logic
- API route validates admin auth before processing
- Consider sending "processed" confirmation email to requester

---

### Story 7.5: Add Featured Analysis Management

As an admin (Mario),
I want to add or remove DAOs from the Featured Analysis list,
So that I can control which DAOs receive ChainSights-commissioned reports.

**Acceptance Criteria:**

**Given** I navigate to `/admin/featured`
**When** the page loads
**Then** two sections display:
  - **Current Featured DAOs:** List of DAOs with "Remove" button
  - **Add New Featured DAO:** Form to add DAO (name, slug, Snapshot space, category)
**And** adding a new DAO:
  - Validates DAO exists on Snapshot.org
  - Inserts into `daos` table with `badge_type = 'featured_analysis'` (new column)
  - Triggers manual GVS calculation for the new DAO
  - Shows success message
**And** removing a DAO:
  - Does NOT delete from database (keep historical data)
  - Sets `badge_type = null` or marks as inactive
  - DAO no longer shown with "Featured Analysis" badge on leaderboard
**And** admin can edit DAO details (name, category, Snapshot space)

**Implementation Notes:**
- Add `badge_type` enum column to `daos` table: null, 'featured_analysis', 'customer_report'
- Form validation: Check Snapshot space ID is valid
- Future: Customer Reports badge set when DAO opts in after purchase

---

**Epic 7 Complete:** 5 stories created covering all 10 admin requirements

---

## Epic 8: Production Readiness & Monitoring

**Goal:** Ensure system meets all quality standards for public launch: performance, security, reliability, accessibility, SEO.

**User Outcome:** Users experience fast page loads, reliable uptime, secure connections, accessible interface, and discoverable content.

**Requirements Covered:** FR75-FR93 + All 119 NFRs (138 requirements total)

---

### Story 8.1: Implement Core Web Vitals Optimization

As a user,
I want pages to load quickly and feel responsive,
So that I have a smooth browsing experience.

**Acceptance Criteria:**

**Given** any public page on the site
**When** performance is measured
**Then** Core Web Vitals targets are met (p95):
  - **LCP (Largest Contentful Paint):** <2.0s (NFR-PERF-2)
  - **FCP (First Contentful Paint):** <1.5s (NFR-PERF-1)
  - **TTI (Time to Interactive):** <3.0s (NFR-PERF-3)
  - **CLS (Cumulative Layout Shift):** <0.1 (NFR-PERF-4)
  - **TBT (Total Blocking Time):** <200ms (NFR-PERF-5)
**And** optimization techniques applied:
  - ISR caching (1-hour revalidation) for `/rankings` page
  - Server Components by default (minimize JS bundle)
  - Next.js Image component for all images (automatic WebP conversion)
  - Font optimization (Montserrat + Inter subsets loaded efficiently)
  - Code splitting (dynamic imports for heavy components)
**And** JavaScript bundle <150KB gzipped (NFR-PERF-10)
**And** CSS bundle <30KB gzipped (NFR-PERF-11)
**And** Lighthouse CI configured to fail builds if scores drop below thresholds
**And** Vercel Analytics tracks real-user Core Web Vitals

**Implementation Notes:**
- Add Lighthouse CI: `npm install -D @lhci/cli`
- Configure: `.lighthouserc.json` with performance budgets
- Monitor with Vercel Analytics dashboard (already configured)
- Use `next/image` for all images (ARCH-TECH-6)

---

### Story 8.2: Configure Security Headers and Rate Limiting

As a security-conscious operator,
I want the application protected from common web vulnerabilities,
So that user data and system integrity are maintained.

**Acceptance Criteria:**

**Given** any request to the application
**When** security measures are evaluated
**Then** the following protections are active:
  - **HTTPS enforced:** All traffic over HTTPS with HSTS header (NFR-SEC-1)
  - **Content Security Policy:** Restrict script sources, prevent inline scripts (NFR-SEC-5)
  - **Security headers:** X-Content-Type-Options, X-Frame-Options, X-XSS-Protection (NFR-SEC-15)
  - **Rate limiting:** 100 requests/minute per IP via Vercel Edge Middleware (NFR-SEC-4, FR81)
  - **CORS:** API endpoints restricted to same-origin (NFR-SEC-6)
  - **Input sanitization:** Zod validation on all forms (NFR-SEC-3, Epic 3)
**And** rate limiting returns 429 status with "Too Many Requests" message
**And** security headers configured in `next.config.js` or middleware
**And** no sensitive data logged (API keys, tokens, user emails) per NFR-SEC-16
**And** database credentials stored in environment variables (never hardcoded)
**And** Dependabot alerts enabled for npm package vulnerabilities (NFR-SEC-14)

**Implementation Notes:**
- Security headers: Add to `next.config.js` headers config
- Rate limiting: Implement in `src/middleware.ts` using Vercel KV or in-memory store
- CSP: `Content-Security-Policy: default-src 'self'; script-src 'self'`
- Monitor security headers with: https://securityheaders.com

---

### Story 8.3: Set Up Monitoring and Error Tracking

As a developer/operator,
I want to be alerted when errors occur or performance degrades,
So that I can fix issues before they impact many users.

**Acceptance Criteria:**

**Given** the application is running in production
**When** errors occur or metrics degrade
**Then** monitoring tools capture and alert:
  - **Error tracking:** Sentry captures JavaScript errors, API errors, unhandled exceptions
  - **Uptime monitoring:** UptimeRobot pings `/rankings` every 5 minutes (NFR-REL-11)
  - **Performance monitoring:** Vercel Analytics tracks Core Web Vitals (FR87)
  - **Database monitoring:** Neon dashboard shows query performance, connection pool usage
  - **Alerting rules:**
    - Sentry: Alert if >10 errors/hour (NFR-REL-12)
    - UptimeRobot: Alert if downtime >2 minutes
    - Custom: Alert if LCP increases >20% week-over-week (FR93, NFR-REL-14)
**And** Sentry initialized in `src/app/layout.tsx` with DSN from environment variable
**And** source maps uploaded to Sentry for readable stack traces
**And** UptimeRobot configured with email/Slack notifications
**And** monitoring respects user privacy (no PII captured)

**Implementation Notes:**
- Install Sentry: `npm install @sentry/nextjs`
- Configure: `sentry.client.config.ts`, `sentry.server.config.ts`
- UptimeRobot free tier: https://uptimerobot.com
- Neon dashboard: Built-in, no setup needed

---

### Story 8.4: Implement SEO Optimizations

As a marketer/founder (Mario),
I want the site to rank well in search engines,
So that organic traffic discovers the leaderboard.

**Acceptance Criteria:**

**Given** any public page
**When** search engines crawl the site
**Then** SEO optimizations are in place:
  - **Sitemap.xml:** Auto-generated at `/sitemap.xml` including all public pages (FR82, NFR-SEO-3)
  - **Robots.txt:** At `/robots.txt` allowing crawlers, disallowing `/admin/*` (FR83, NFR-SEO-1)
  - **Canonical URLs:** All pages have `<link rel="canonical">` to prevent duplicates (FR84, NFR-SEO-11)
  - **Structured data:** JSON-LD for leaderboard (ItemList schema) and methodology (Article schema) (FR85-86, NFR-SEO-4)
  - **Meta descriptions:** Unique, compelling descriptions for each page (50-160 chars)
  - **Open Graph tags:** Implemented in Epic 5 (NFR-SEO-6-7)
  - **Last Modified:** Sitemap includes `<lastmod>` reflecting actual content updates (NFR-SEO-10)
**And** pages are server-rendered (no SPA routing issues)
**And** no `noindex` tags on public pages (NFR-SEO-2)
**And** URL structure is clean: `/rankings`, `/rankings/methodology` (NFR-SEO-8)
**And** page titles follow pattern: "[Page Name] | ChainSights"
**And** tested with Google Search Console after launch

**Implementation Notes:**
- Sitemap: Use Next.js 14 `sitemap.ts` route
- Robots: Create `public/robots.txt`
- Structured data: Add JSON-LD script tags in page components
- Submit sitemap to Google Search Console after launch

---

### Story 8.5: Configure Backup and Recovery Procedures

As an operator,
I want automated backups and ability to rollback deployments,
So that data is protected and incidents can be quickly resolved.

**Acceptance Criteria:**

**Given** the application is in production
**When** backup and recovery capabilities are reviewed
**Then** the following protections exist:
  - **Database backups:** Neon automatic daily backups with 30-day retention (NFR-REL-15)
  - **Point-in-time recovery:** Neon supports restore to any point in last 30 days
  - **Code versioning:** Git history on GitHub/GitLab allows rollback to any commit (NFR-REL-16)
  - **Deployment rollback:** Vercel 1-click rollback to previous deployment in dashboard (NFR-REL-17)
  - **Environment variables:** Backed up in password manager or secure documentation
  - **Recovery procedures documented:** `docs/operations/disaster-recovery.md` with step-by-step instructions
**And** backup verification: Monthly test restore from backup to staging environment
**And** RTO (Recovery Time Objective): <1 hour for critical issues
**And** RPO (Recovery Point Objective): <24 hours data loss maximum (ideally <1 hour with Neon)
**And** emergency contacts documented for Vercel, Neon, critical services

**Implementation Notes:**
- Neon backups: Built-in, verify in dashboard
- Document rollback procedures for team
- Consider staging environment for testing restores
- Test rollback procedure at least once before launch

---

### Story 8.6: Ensure GDPR and Privacy Compliance

As a legal-compliant operator,
I want the application to respect user privacy and meet GDPR requirements,
So that we avoid legal issues and build user trust.

**Acceptance Criteria:**

**Given** the application collects any user data
**When** GDPR compliance is audited
**Then** the following requirements are met:
  - **Data minimization:** Only collect consented emails for reports/opt-outs (NFR-SEC-9)
  - **Right to erasure:** Opt-out mechanism provides 24-hour removal (NFR-SEC-10, Epic 4)
  - **Data retention:** Emails deleted 90 days after report delivery unless marketing opt-in (NFR-SEC-11)
  - **Cookie consent:** Analytics cookies disclosed in privacy policy (NFR-SEC-12)
  - **Privacy policy:** Comprehensive policy at `/privacy` (NFR-SEC-13, Epic 3)
  - **No tracking across sites:** No third-party tracking pixels
  - **Data processing agreement:** If using third-party processors (Stripe, Resend), ensure DPAs in place
**And** user can request data export (email hello@chainsights.one)
**And** user can request data deletion (via opt-out form or email)
**And** cookie banner NOT required if only using Vercel Analytics (first-party, no consent needed in EU)
**And** GDPR compliance documented in privacy policy

**Implementation Notes:**
- Review third-party services for GDPR compliance
- Stripe and Resend are GDPR-compliant processors
- Automate email deletion with cron job (90-day cleanup)
- No user accounts = simpler GDPR compliance

---

### Story 8.7: Implement Accessibility Testing

As an accessibility advocate,
I want the site to be tested for WCAG 2.1 Level AA compliance,
So that all users can access the leaderboard regardless of ability.

**Acceptance Criteria:**

**Given** the site is built with accessibility features (Epic 2)
**When** accessibility is tested
**Then** the following testing occurs:
  - **Automated testing:** axe-core in Playwright E2E tests catches 60% of issues (NFR-ACCESS-16)
  - **Manual screen reader testing:** NVDA (Windows) + VoiceOver (macOS) tested on key pages (NFR-ACCESS-17)
  - **Keyboard navigation testing:** All interactive elements accessible via keyboard only
  - **Color contrast testing:** WebAIM Contrast Checker verifies all text meets 4.5:1 or 7:1 ratios
  - **Visual testing:** Test at 200% zoom, verify no content loss
  - **Touch target testing:** Verify all buttons/links meet 44×44px minimum on mobile
**And** accessibility statement page exists at `/accessibility` (NFR-ACCESS-18)
**And** statement includes:
  - WCAG 2.1 Level AA compliance goal
  - Known issues and workarounds
  - Contact for accessibility feedback: hello@chainsights.one
  - Last audit date
**And** accessibility issues logged and prioritized for fixes
**And** aim for zero critical/serious axe-core violations before launch

**Implementation Notes:**
- Add axe-core to Playwright: `npm install -D @axe-core/playwright`
- Example test: `tests/e2e/accessibility.spec.ts`
- Screen reader testing: Manual, budget ~2 hours
- Document testing results in `/docs/accessibility-audit.md`

---

### Story 8.8: Configure Analytics and Conversion Tracking

As a product manager (Mario),
I want detailed analytics on user behavior and conversion funnel,
So that I can optimize the product for growth.

**Acceptance Criteria:**

**Given** Vercel Analytics is integrated (Epic 0)
**When** analytics are reviewed
**Then** the following metrics are tracked:
  - **Page views:** All public pages (FR88)
  - **Core Web Vitals:** LCP, FCP, CLS automatically tracked (FR87)
  - **Custom events** (from Epic 5):
    - `cta_click`: Track report CTA conversions (FR89)
    - `social_share`: Track social sharing rate (FR90)
    - `row_expand`: Track engagement with DAO details
    - `methodology_view`: Track methodology page visits
  - **Conversion funnel:** Leaderboard view → Row expand → CTA click → Order (manual tracking initially)
  - **Opt-out rate:** Calculate from opt-out requests vs total DAOs ranked (FR91)
  - **Repeat visitors:** Track returning users (FR92)
**And** analytics dashboard accessible in Vercel (web interface)
**And** custom event code in `src/lib/analytics.ts`
**And** no PII tracked (no emails, IPs stored beyond Vercel's standard practice)
**And** analytics data reviewed weekly to inform product decisions

**Implementation Notes:**
- Vercel Analytics Web interface: https://vercel.com/docs/analytics
- Events already implemented in Epic 5
- Create weekly report template for Mario to review metrics

---

**Epic 8 Complete:** 8 stories created covering production readiness (many NFRs addressed across multiple stories)

---

## Epic 9: UX Enhancements (MVP+1 Week)

**Goal:** Improve first-time user comprehension, mobile interaction, and visual clarity based on UX validation findings.

**User Outcome:** First-time visitors understand leaderboard faster, mobile users tap entire cards naturally, severity indicators are instantly clear.

**Requirements Covered:** UX-P1-1, UX-P1-2, UX-P1-3, UX-P2-1, UX-P2-2, UX-P2-3 (6 requirements)

---

### Story 9.1: Make Entire Mobile Card Tappable with Chevron

As a mobile user,
I want to tap anywhere on a DAO card to expand it,
So that interaction feels natural and matches my expectations.

**Acceptance Criteria:**

**Given** I view the leaderboard on mobile
**When** I tap anywhere on a DAO row
**Then** the row expands to show component breakdown
**And** chevron icon (▼) displays in top-right corner of card
**And** chevron rotates 180° to (▲) when expanded
**And** entire card has cursor-pointer on desktop, active:bg-gray-50 on tap
**And** keyboard navigation still works (Enter to toggle)
**And** screen reader announces expanded state change
**And** mobile user testing shows >90% successful expansion rate (UX-P1-1 acceptance criteria)

**Implementation Notes:**
- Update `src/components/ExpandableDAORow.tsx`
- Add `onClick` to outer container div instead of just button
- Icon: Use Lucide React `ChevronDown` with rotation transform
- Reference UX validation report for implementation details

---

### Story 9.2: Add First-Visit Onboarding Banner

As a first-time visitor,
I want a brief explanation of what I'm looking at,
So that I can understand the leaderboard within 30 seconds.

**Acceptance Criteria:**

**Given** I visit `/rankings` for the first time
**When** the page loads
**Then** an info banner displays at top of leaderboard:
  - Icon: 💡 (light bulb emoji)
  - Text: "New here? This leaderboard ranks DAOs on governance health across 5 criteria. Click any DAO to see component scores and understand why they ranked that way."
  - Two CTAs: "Got it" (dismiss) and "Learn More →" (to methodology page)
**And** banner styled with aqua accent border, light background
**And** clicking "Got it" dismisses banner and stores flag in localStorage: `rankings-visited = 'true'`
**And** banner never shows again for returning visitors (localStorage check)
**And** banner is accessible (can be dismissed with keyboard Escape key)
**And** A/B test shows bounce rate decreases by >10% with banner enabled (UX-P1-2 acceptance criteria)

**Implementation Notes:**
- Component: `src/components/FirstVisitBanner.tsx` (Client Component with useEffect)
- Use `localStorage` for persistence
- Reference UX validation report for exact copy and styling

---

### Story 9.3: Add Visual Severity Indicators to Top 5 Control

As a user scanning the leaderboard,
I want to quickly understand if "Top 5 Control: 52%" is concerning or not,
So that I don't have to mentally calculate severity.

**Acceptance Criteria:**

**Given** a DAO row displays "Top 5 Control" percentage
**When** the percentage is rendered
**Then** visual indicators appear based on thresholds:
  - ✓ (green checkmark): <30% - Well-distributed
  - ⚠️ (yellow warning): 30-50% - Moderate concentration
  - 🔶 (orange diamond): 50-75% - High concentration
  - 🔴 (red circle): >75% - Extreme concentration
**And** icon displays before the percentage: "✓ 28%" or "🔴 76%"
**And** `aria-label` announces severity for screen readers: "Well-distributed, 28%"
**And** icons distinguishable by shape (color-blind accessible)
**And** user testing shows >80% correctly identify severity at a glance (UX-P1-3 acceptance criteria)
**And** icons visible and readable on mobile layout

**Implementation Notes:**
- Component: `src/components/Top5Control.tsx`
- Use emoji unicode or icon font (Lucide React)
- Calculate "Top 5 Control" from Nakamoto coefficient (PDI component, Epic 1)
- Reference UX validation report for threshold details

---

### Story 9.4: Add Empty State CTA for Low DAO Count Categories

As a user filtering by category with few results,
I want to see a helpful CTA instead of an empty table,
So that I can request coverage for missing DAOs.

**Acceptance Criteria:**

**Given** I filter leaderboard by category (e.g., "Public Goods")
**When** the category has <5 DAOs
**Then** an empty state message displays below the table:
  - Icon: 📊 (bar chart emoji)
  - Heading: "Want to see your DAO ranked here?"
  - Text: "We're actively expanding our [Category] coverage. Request a free analysis for your DAO to appear on this leaderboard."
  - CTA button: "Request Free Analysis →"
**And** clicking button goes to lead capture form (future: `/request-analysis`, MVP: email link)
**And** message personalizes by category: "Public Goods coverage"
**And** tracks conversions from empty state CTA with analytics event
**And** empty state does NOT show if ≥5 DAOs in category (UX-P2-1)

**Implementation Notes:**
- Component: `src/components/EmptyStateLeadGen.tsx`
- Check DAO count after filtering
- Lead capture form can be MVP email link initially

---

### Story 9.5: Add Sticky "Last Updated" Header on Scroll

As a user scrolling through the leaderboard,
I want to still see when data was updated,
So that I maintain context about freshness while browsing.

**Acceptance Criteria:**

**Given** I scroll down the leaderboard page
**When** I scroll past 200px from top
**Then** a sticky header appears at top of viewport:
  - Text: "DAO Rankings • Updated 2 hours ago"
  - Expandable dropdown (chevron icon) shows full details on click:
    - Last calculation: [Full timestamp]
    - Next update: Sunday midnight UTC
    - Link: "See methodology →"
**And** sticky header has backdrop blur effect (bg-navy/95 backdrop-blur-sm)
**And** header disappears when scrolled back to top (<200px)
**And** header does not cover leaderboard content (z-index managed)
**And** mobile-friendly: Collapses to icon + timestamp on small screens (UX-P2-2)

**Implementation Notes:**
- Component: `src/components/StickyUpdateBanner.tsx` (Client Component with scroll listener)
- Use `useState` and `useEffect` to track scroll position
- Reference UX validation report for styling details

---

### Story 9.6: Add Historical Trend Sparklines (Optional Post-MVP)

As an engaged user,
I want to see a mini chart of score trends,
So that I can visualize DAO governance improvement or decline over time.

**Acceptance Criteria:**

**Given** a DAO has ≥4 weeks of historical snapshots
**When** the DAO row is displayed (collapsed or expanded state)
**Then** a small sparkline (20px tall) shows 4-8 weeks of GVS history:
  - Line chart with subtle gradient
  - Color: Green for uptrend, red for downtrend, gray for stable
  - Tooltip on hover shows exact weekly values
**And** sparkline is accessible to screen readers (data table fallback or aria-label)
**And** mobile layout includes sparklines (may stack or inline depending on space)
**And** if <4 weeks history, show "New" badge instead of sparkline
**And** user engagement increases (measured via time-on-page, repeat visits) per UX-P2-3

**Implementation Notes:**
- Library: `react-sparklines` or lightweight alternative
- Component: `src/components/ScoreTrendSparkline.tsx`
- Fetch historical snapshots (last 8 weeks) with DAO data
- Optional/Future: Implement after MVP if time allows

---

**Epic 9 Complete:** 6 stories created covering all UX enhancement requirements

---

## Summary: All Epics Complete

**Total Story Count: 51 stories across 10 epics**

- Epic 0: 5 stories (Project Foundation)
- Epic 1: 8 stories (GVS Calculation Engine)
- Epic 2: 10 stories (Public Rankings Leaderboard)
- Epic 3: 4 stories (Methodology & Legal)
- Epic 4: 3 stories (Opt-Out Management)
- Epic 5: 4 stories (Report CTAs & Social)
- Epic 6: 2 stories (Weekly Automation)
- Epic 7: 5 stories (Admin Dashboard)
- Epic 8: 8 stories (Production Readiness)
- Epic 9: 6 stories (UX Enhancements)

**All 228 requirements covered:**
- 93 Functional Requirements (FR1-FR93)
- 119 Non-Functional Requirements (NFR categories)
- 8 Architecture Requirements (ARCH-SETUP + ARCH-TECH)
- 8 UX Validation Requirements (UX-P0, UX-P1, UX-P2)

**Implementation-ready for development teams with clear acceptance criteria using Given/When/Then format for every story.**
