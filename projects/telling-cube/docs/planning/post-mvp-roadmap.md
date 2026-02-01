# Post-MVP Roadmap

> Planning document for tellingCube development after v1.1.0 launch
> Last updated: December 2025

## Overview

This document outlines the development roadmap after the MVP launch. Items are organized by priority and complexity. The first priority is **Authentication** to replace the current email-lookup approach.

---

## Phase 1: Foundation (Priority: HIGH)

### 1.1 Authentication System ⭐ FIRST PRIORITY

**Why:** Current email-lookup is a workaround. Proper auth enables all future features.

**Scope:**
- [ ] User registration (email + password)
- [ ] Login / Logout
- [ ] Password reset flow
- [ ] Link Stripe purchases to user accounts
- [ ] Migrate existing Stripe customers to user accounts
- [ ] Session management

**Technical Approach:**
- **Recommended:** NextAuth.js (integrates well with Prisma + PostgreSQL + Stripe)
- **Alternatives:** Clerk (faster setup), Supabase Auth (if we migrate DB)

**Database Changes:**
```prisma
model User {
  id            String    @id @default(uuid())
  email         String    @unique
  passwordHash  String?
  name          String?
  tier          String?   // 'supporterS' | 'supporterM' | etc.
  stripeCustomerId String? @unique
  scenarios     Scenario[]
  createdAt     DateTime  @default(now())
  updatedAt     DateTime  @updatedAt
}

model Scenario {
  // ... existing fields
  userId        String?   // NEW: Replace ipAddress filtering
  user          User?     @relation(fields: [userId], references: [id])
}
```

**Stories:**
- [ ] Story: Database schema for User model
- [ ] Story: NextAuth.js setup with credentials provider
- [ ] Story: Registration page UI
- [ ] Story: Login page UI
- [ ] Story: Link Stripe webhook to create/update users
- [ ] Story: Migrate "My Scenarios" from IP to userId
- [ ] Story: Protected routes middleware

---

### 1.2 User Dashboard (SaaS Platform)

**Why:** Listed in "What's Next" as priority. Enables scenario management.

**Scope:**
- [ ] Dashboard page after login
- [ ] List user's scenarios with status
- [ ] Delete old scenarios
- [ ] View generation history
- [ ] Account settings

**Dependencies:** Authentication (1.1)

---

## Phase 2: Content Expansion

### 2.1 HR View

**Why:** Listed in "What's Next". Completes the business data story.

**Scope:**
- [ ] Headcount trends chart
- [ ] Department breakdown
- [ ] Payroll analysis
- [ ] Employee turnover metrics
- [ ] Hiring timeline

**Technical:**
- Events already contain employee data (HR_HIRE, PAYROLL, etc.)
- Need new aggregation queries
- New view components

---

### 2.2 More Industries / Sectors

**Why:** Listed in "What's Next". Expands market reach.

**Current:** Consumer Staples, Industrials, Financials

**Candidates for brainstorming:**
- [ ] Energy
- [ ] Healthcare
- [ ] Technology (not just startups)
- [ ] Real Estate
- [ ] Utilities
- [ ] Materials

**Per sector needs:**
- Industry-specific event types
- Realistic KPIs and metrics
- Appropriate terminology
- Sample company names

---

### 2.3 Scenario Comparison

**Why:** Listed in "What's Next". High value for training.

**Scope:**
- [ ] Select 2-3 scenarios to compare
- [ ] Side-by-side metrics
- [ ] Variance highlighting
- [ ] Export comparison report

---

## Phase 3: Export & Integration

### 3.1 Enhanced Exports

**Why:** Listed in "What's Next". Business users need Excel.

**Scope:**
- [ ] Excel (.xlsx) export with formatting
- [ ] PDF reports with charts
- [ ] Customizable export templates

**Technical:**
- xlsx library for Excel generation
- pdf-lib or Puppeteer for PDF

---

### 3.2 Controlling View

**Why:** Listed in "What's Next" (Later). Completes finance story.

**Scope:**
- [ ] Cost center drill-down
- [ ] Budget vs Actual analysis
- [ ] Variance explanations
- [ ] Allocation visualization

---

## Phase 4: Advanced Features

### 4.1 Multi-Year Scenarios

**Why:** Listed in "What's Next" (Later). Enables forecasting training.

**Scope:**
- [ ] 3-5 year projections
- [ ] Growth modeling
- [ ] Seasonality patterns
- [ ] Economic cycle simulation

---

### 4.2 Custom Parameters

**Why:** Listed in "What's Next" (Later). User control.

**Scope:**
- [ ] Adjust growth rates
- [ ] Set seasonality patterns
- [ ] Company size slider
- [ ] Market conditions

---

## Phase 5: Vision Features

> These are long-term goals. Priorities will shift based on user feedback.

### 5.1 tellingdash
- Advanced visualization suite
- Custom dashboards
- Presentation exports
- IBCS© compliance tools

### 5.2 Enterprise Features
- Team workspaces
- Role-based access control
- Audit logs
- SSO integration

### 5.3 White-Label
- Embed in training platforms
- Custom branding
- API for partners

### 5.4 Integrations
- Power BI connector
- Tableau connector
- SAP data import
- Excel add-in

### 5.5 Custom Templates
- Industry-specific templates
- User-created templates
- Template marketplace

### 5.6 AI Customization
- Natural language adjustments ("make Q4 sales 20% higher")
- What-if scenarios
- Anomaly injection

---

## Brainstorming Parking Lot

> Ideas to discuss in future sessions

- [ ] Mobile app / PWA
- [ ] Collaboration features (share scenarios)
- [ ] Scenario annotations / notes
- [ ] Training mode with guided exercises
- [ ] Certification integration
- [ ] Gamification elements
- [ ] Community templates
- [ ] Benchmark comparisons
- [ ] Real company data anonymization
- [ ] LMS (Learning Management System) integration
- [ ] Multi-language support (German, French, etc.)
- [ ] Accessibility improvements (WCAG compliance)

---

## Technical Debt & Infrastructure

> Items to address alongside features

- [ ] Migrate to new GitHub URL (masem-at/telling-cube)
- [ ] Set up staging environment
- [ ] Add end-to-end tests (Playwright)
- [ ] Performance monitoring (Vercel Analytics)
- [ ] Error tracking (Sentry)
- [ ] Database backups automation
- [ ] API rate limiting refinement
- [ ] CDN for static assets
- [ ] Image optimization

---

## Version Planning

| Version | Focus | Key Features |
|---------|-------|--------------|
| v1.1.0 | Launch | Paid user flow (current) |
| v1.2.0 | Auth | User registration, login, dashboard |
| v1.3.0 | Content | HR View, more sectors |
| v1.4.0 | Export | Excel, PDF, comparison |
| v2.0.0 | Platform | Multi-year, custom params, enterprise |

---

## Decision Log

| Date | Decision | Rationale |
|------|----------|-----------|
| 2025-12 | NextAuth.js for auth | Integrates with existing Prisma + Stripe stack |
| 2025-12 | Auth is first priority | Foundation for all user-related features |

---

## Notes

- Priorities may shift based on Founding Member feedback
- Each phase should have QA gate before release
- Consider beta testing with select users for major features
