---
stepsCompleted: [1, 2, 3, 4]
inputDocuments:
  - docs/analysis/product-brief-admin-dashboard-2026-01-22.md
workflowType: 'epics-stories'
lastStep: 4
project_name: 'chainsights-admin-dashboard'
user_name: 'Mario'
date: '2026-01-22'
status: complete
---

# ChainSights Admin Dashboard - Epic Breakdown

## Overview

This document provides the complete epic and story breakdown for the ChainSights Admin Dashboard Redesign, decomposing the requirements from the Product Brief into implementable stories.

## Requirements Inventory

### Functional Requirements

| FR | Requirement | Source |
|----|-------------|--------|
| FR1 | System shall display a left sidebar with navigation links to all major sections (Overview, Reports, Actions, Content) | MVP Feature 1 |
| FR2 | System shall display a status overview panel at the top showing key health indicators (last GVS run time, failure count, job status) | MVP Feature 2 |
| FR3 | System shall provide a dedicated Reports section for report generation and management | MVP Feature 3 |
| FR4 | System shall provide an Actions section grouping operational triggers (Recalculate GVS, Test Report) in card-based layout | MVP Feature 4 |
| FR5 | System shall provide a Content section for feature list management and Featured DAOs table | MVP Feature 5 |
| FR6 | All major sections shall be reachable in ≤2 clicks from the sidebar | Success Criteria |
| FR7 | Report generation shall complete in ≤3 clicks from admin entry | Success Criteria |

### Non-Functional Requirements

| NFR | Requirement | Source |
|-----|-------------|--------|
| NFR1 | Health status shall be visible in <2 seconds without scrolling | Success Metrics |
| NFR2 | Dashboard shall follow Vercel/GitHub UX conventions for familiar patterns | UX Research |
| NFR3 | Layout shall use shadcn/ui card components for visual consistency | Tech Stack |
| NFR4 | Structure shall be scalable for adding future features without clutter | Business Objective |

### Additional Requirements

- Existing admin page at `/admin` needs restructuring (not new page)
- Must preserve all existing functionality (recalculate, test report, feature list, PDF display, source display)
- Authentication already exists (reuse existing admin auth)
- Tech stack: Next.js 14, React, shadcn/ui, Tailwind CSS

### FR Coverage Map

| FR | Epic | Description |
|----|------|-------------|
| FR1 | Epic 1 | Sidebar navigation implementation |
| FR2 | Epic 1 | Status overview panel |
| FR3 | Epic 2 | Reports section |
| FR4 | Epic 2 | Actions section with cards |
| FR5 | Epic 3 | Content management section |
| FR6 | Epic 1 | ≤2 click navigation (sidebar structure) |
| FR7 | Epic 2 | ≤3 click report generation |

## Epic List

### Epic 1: Dashboard Layout & Navigation

**User Outcome:** Admin can navigate all dashboard sections efficiently via a sidebar and see system health at a glance without scrolling.

**FRs covered:** FR1, FR2, FR6
**NFRs addressed:** NFR1, NFR2, NFR3, NFR4

**Implementation Notes:**
- Restructures existing `/admin` page with left sidebar
- Adds Status Overview panel at top
- Establishes scalable layout foundation
- Standalone: Provides complete navigation even before sections have content

---

### Epic 2: Operations Hub (Reports & Actions)

**User Outcome:** Admin can generate reports in ≤3 clicks and trigger operational tasks (Recalculate GVS, Test Report) from a clear card-based interface.

**FRs covered:** FR3, FR4, FR7

**Implementation Notes:**
- Migrates existing report generation to dedicated Reports section
- Migrates existing triggers to Actions section with card layout
- Preserves all existing functionality
- Standalone: Works with Epic 1's navigation structure

---

### Epic 3: Content Management

**User Outcome:** Admin can manage feature lists and Featured DAOs table in a dedicated Content section.

**FRs covered:** FR5

**Implementation Notes:**
- Migrates existing Feature List management
- Migrates existing Featured DAOs table
- Standalone: Works independently within the new layout

---

### Epic 4: Daily GVS Monitoring & Historical Tracking

**User Outcome:** Admin receives immediate alerts when DAO scores change significantly, with daily data collection enabling future trend analysis and charts.

**FRs covered:** FR8 (new), FR9 (new), FR10 (new)
**NFRs addressed:** NFR1, NFR4

**Business Context:**
- Critical gap discovered: Lido score dropped 7.5 → 5.8 between Saturday and Wednesday, only caught by manual recalculation
- Without daily monitoring, significant governance changes go unnoticed for up to 7 days
- Enables future features: trend charts, public ticker, anomaly reports

**Implementation Notes:**
- Converts weekly cron to daily calculation
- Adds monthly aggregation for long-term history
- Implements anomaly detection with configurable threshold
- Adds admin dashboard alert card for immediate visibility
- Standalone: Can be implemented independently of UI restructuring

**New Functional Requirements:**

| FR | Requirement | Source |
|----|-------------|--------|
| FR8 | System shall calculate GVS scores daily and store with snapshot_type='daily' | Epic 4 |
| FR9 | System shall detect score changes > configurable threshold and alert admin via Slack/email | Epic 4 |
| FR10 | System shall aggregate monthly scores (last day, average, min, max) on 1st of each month | Epic 4 |

---

## Stories

### Epic 1: Dashboard Layout & Navigation

#### Story 1.1: Create Admin Dashboard Shell with Sidebar Navigation

As an **admin**,
I want a **sidebar navigation layout** replacing the top header links,
So that I can **navigate between sections efficiently in ≤2 clicks**.

**Acceptance Criteria:**

**Given** I am authenticated as admin
**When** I visit `/admin`
**Then** I see a left sidebar with navigation links: Overview, Reports, Actions, Content
**And** the sidebar is always visible (desktop) or collapsible (mobile)
**And** each section is highlighted when active
**And** the main content area displays the active section

**Given** I click a sidebar navigation item
**When** the page updates
**Then** I remain on `/admin` (client-side navigation within dashboard)
**And** the corresponding section content is displayed
**And** all navigation completes in ≤2 clicks from any section

**Technical Notes:**
- Use shadcn/ui `Sheet` for mobile sidebar, fixed sidebar for desktop
- Follow Vercel dashboard pattern: left sidebar, main content right
- Preserve existing auth check (`isAdminAuthenticated`)

---

#### Story 1.2: Implement Status Overview Panel

As an **admin**,
I want a **status overview panel at the top of the dashboard**,
So that I can **instantly see system health without scrolling**.

**Acceptance Criteria:**

**Given** I am on the admin dashboard
**When** the page loads
**Then** I see a status overview panel at the top showing:
  - Last GVS run time and status (success/failed)
  - Number of failed jobs in last 7 days
  - Pending opt-out requests count
  - Total active DAOs count
**And** the panel is visible in <2 seconds without scrolling
**And** each indicator uses color coding (green=healthy, yellow=attention, red=critical)

**Given** any indicator shows non-healthy status
**When** I view the status panel
**Then** I can click the indicator to navigate to the relevant section

**Technical Notes:**
- Reuse data from `getRecentJobLogs()` and `getPendingOptOutRequests()`
- Use shadcn/ui cards with status badges
- Glanceable design - minimal text, clear icons

---

#### Story 1.3: Migrate Overview Section Content

As an **admin**,
I want the **Overview section** to show key stats and recent activity,
So that I can **get a quick summary of the system state**.

**Acceptance Criteria:**

**Given** I am on the Overview section (default on dashboard load)
**When** the content loads
**Then** I see:
  - Stats cards: Total Orders, Ready for Review, Delivered, Pending, Share Claims
  - Recent GVS Jobs (last 5)
  - Pending Opt-Out Requests (last 10)
**And** each card links to its detailed section

**Given** I have pending share claims
**When** I view the Overview section
**Then** the Share Claims card is highlighted with count

**Technical Notes:**
- Migrate existing stats cards and system status sections
- Keep Weekly GVS Jobs and Opt-Out Requests displays

---

### Epic 2: Operations Hub (Reports & Actions)

#### Story 2.1: Create Reports Section with Order Management

As an **admin**,
I want a **dedicated Reports section** for viewing and managing orders,
So that I can **find and process reports efficiently**.

**Acceptance Criteria:**

**Given** I navigate to the Reports section via sidebar
**When** the section loads
**Then** I see the orders list with search and sort functionality
**And** I can filter by status (pending, processing, delivered, etc.)
**And** I can switch between list view and tree view (grouped by space)
**And** clicking an order opens the report detail at `/admin/reports/[id]`

**Given** I use the search bar
**When** I type a search term
**Then** orders are filtered by email, DAO name, or snapshot space
**And** results update in real-time

**Technical Notes:**
- Migrate existing `SearchSortBar` and orders list from main page
- Preserve existing order display logic and status badges
- Keep link to individual report pages

---

#### Story 2.2: Create Actions Section with Card-Based Layout

As an **admin**,
I want an **Actions section grouping operational triggers**,
So that I can **quickly execute common tasks from a clear interface**.

**Acceptance Criteria:**

**Given** I navigate to the Actions section via sidebar
**When** the section loads
**Then** I see card-based actions:
  - **Recalculate GVS**: Single DAO dropdown + Recalculate All button
  - **Test Report**: Form to generate test report for any DAO
**And** each action card has clear title, description, and controls

**Given** I trigger "Recalculate All"
**When** the action completes
**Then** I see success/error feedback
**And** I can view the job in the Jobs section

**Given** I use "Test Report" form
**When** I submit a valid snapshot space
**Then** a test report is generated (≤3 clicks: Actions → fill form → submit)

**Technical Notes:**
- Migrate `RecalculateControls` component
- Migrate `TestReportForm` component
- Use shadcn/ui cards with consistent styling

---

#### Story 2.3: Add Quick Navigation Links in Actions

As an **admin**,
I want **quick links to specialized admin pages**,
So that I can **access Jobs, Insights, and Opt-Outs management**.

**Acceptance Criteria:**

**Given** I am in the Actions section
**When** I view the section
**Then** I see quick link cards to:
  - `/admin/jobs` - Job history and logs
  - `/admin/insights` - Analytics dashboard
  - `/admin/opt-outs` - Opt-out request processing
**And** each card shows relevant count/status badge

**Given** I click a quick link card
**When** the page navigates
**Then** I am taken to the corresponding admin sub-page

**Technical Notes:**
- These remain separate pages (not embedded in dashboard)
- Show pending counts as badges (e.g., "3 pending opt-outs")

---

### Epic 3: Content Management

#### Story 3.1: Create Content Section with Featured DAOs Management

As an **admin**,
I want a **Content section for managing Featured DAOs**,
So that I can **curate which DAOs appear as featured on the rankings page**.

**Acceptance Criteria:**

**Given** I navigate to the Content section via sidebar
**When** the section loads
**Then** I see the Featured DAOs management interface
**And** I can view currently featured DAOs with their badges
**And** I can add/remove DAOs from featured list
**And** changes are reflected on the public rankings page

**Given** I add a DAO to featured list
**When** I confirm the action
**Then** the DAO appears in the featured list with badge type selection
**And** the change is persisted to the database

**Technical Notes:**
- Embed or link to existing `/admin/featured` functionality
- Use existing `getFeaturedDAOs()` and related API routes

---

#### Story 3.2: Add Free Tier Management Link

As an **admin**,
I want **access to Free Tier management** from the Content section,
So that I can **manage free tier allocations alongside other content**.

**Acceptance Criteria:**

**Given** I am in the Content section
**When** I view the section
**Then** I see a card linking to Free Tier management
**And** the card shows current free tier user count

**Given** I click the Free Tier card
**When** the page navigates
**Then** I am taken to `/admin/free-tier`

**Technical Notes:**
- Free Tier remains a separate page
- Show summary stats on the card

---

### Epic 4: Daily GVS Monitoring & Historical Tracking

#### Story 4.1: Convert Weekly GVS Calculation to Daily

As an **admin**,
I want **GVS scores calculated daily** instead of weekly,
So that I can **detect governance changes within 24 hours instead of 7 days**.

**Acceptance Criteria:**

**Given** the daily cron job runs at midnight UTC
**When** the job completes
**Then** new GVS snapshots are created for all non-opted-out DAOs
**And** each snapshot has `snapshot_type='daily'`
**And** the delta calculation compares to the previous day's snapshot
**And** rankings page shows updated scores

**Given** yesterday's snapshot exists for a DAO
**When** today's calculation completes
**Then** the Change column shows the daily delta (not "New")
**And** delta direction (up/down/stable) is calculated correctly

**Technical Notes:**
- Modify `vercel.json`: change schedule from `0 0 * * 0` (weekly) to `0 0 * * *` (daily)
- Add `snapshot_type` column to `gvs_snapshots` table (default: 'daily')
- Update `calculateDelta()` in `storage.ts` to find previous day's snapshot instead of 7-day window
- Reuse existing `executeWeeklyRecalculation()` logic (rename to `executeDailyRecalculation`)

---

#### Story 4.2: Implement Monthly Aggregation Job

As an **admin**,
I want **monthly GVS aggregates** stored automatically,
So that I can **analyze long-term governance trends** without storing 365 daily snapshots.

**Acceptance Criteria:**

**Given** it is the 1st of the month at 1am UTC
**When** the monthly aggregation job runs
**Then** for each DAO with daily snapshots from previous month:
  - A monthly snapshot is created with `snapshot_type='monthly'`
  - `gvs_score` contains the last day's score
  - `monthly_avg_score` contains the average of all daily scores
  - `monthly_min_score` contains the minimum score
  - `monthly_max_score` contains the maximum score
**And** monthly snapshots are never deleted by cleanup jobs

**Given** a DAO has no daily snapshots for the previous month
**When** the aggregation runs
**Then** no monthly snapshot is created for that DAO
**And** a warning is logged

**Technical Notes:**
- New cron job: `/api/jobs/monthly-aggregation` with schedule `0 1 1 * *`
- Add columns to schema: `monthly_avg_score`, `monthly_min_score`, `monthly_max_score` (nullable)
- Query: `SELECT dao_id, AVG(gvs_score), MIN(gvs_score), MAX(gvs_score) FROM gvs_snapshots WHERE snapshot_type='daily' AND snapshot_date >= first_of_prev_month AND snapshot_date < first_of_current_month GROUP BY dao_id`
- Keep 12+ months of monthly snapshots

---

#### Story 4.3: Implement 30-Day Rolling Cleanup Job

As an **admin**,
I want **daily snapshots older than 30 days automatically deleted**,
So that **database storage remains bounded** while keeping recent history.

**Acceptance Criteria:**

**Given** the cleanup job runs daily at 2am UTC
**When** daily snapshots older than 30 days exist
**Then** those snapshots are deleted
**And** monthly snapshots are never deleted (regardless of age)
**And** a log entry records how many snapshots were deleted

**Given** no snapshots older than 30 days exist
**When** the cleanup job runs
**Then** no deletions occur
**And** job completes successfully

**Technical Notes:**
- New cron job: `/api/jobs/snapshot-cleanup` with schedule `0 2 * * *`
- Query: `DELETE FROM gvs_snapshots WHERE snapshot_type='daily' AND snapshot_date < NOW() - INTERVAL '30 days'`
- Add job logging similar to weekly recalculation job

---

#### Story 4.4: Implement Anomaly Detection with Alerts

As an **admin**,
I want **immediate alerts when a DAO's score changes significantly**,
So that I can **investigate unexpected governance changes** before they affect reports or outreach.

**Acceptance Criteria:**

**Given** the daily calculation completes for a DAO
**When** the score change from yesterday exceeds the threshold (default: ±0.5)
**Then** an anomaly is recorded with: DAO name, old score, new score, delta, timestamp
**And** a Slack notification is sent with the anomaly details
**And** an email alert is sent to admin

**Given** anomaly threshold is set to 0.5
**When** Lido's score changes from 7.5 to 5.8 (delta = -1.7)
**Then** this triggers an anomaly alert
**And** the alert clearly shows: "Lido: 7.5 → 5.8 (-1.7) ⚠️ SIGNIFICANT CHANGE"

**Given** the threshold is configurable via environment variable
**When** `GVS_ANOMALY_THRESHOLD=0.7` is set
**Then** only changes > ±0.7 trigger alerts

**Technical Notes:**
- Add `GVS_ANOMALY_THRESHOLD` env var (default: 0.5)
- Create `detectAnomalies()` function in new file `src/lib/gvs/anomaly-detection.ts`
- Call after each DAO calculation in daily job
- Reuse existing `sendSlackAlert()` and `sendEmailAlert()` functions
- Consider: store anomalies in job_logs JSON field or new `gvs_anomalies` table

---

#### Story 4.5: Add Anomaly Alert Card to Admin Dashboard

As an **admin**,
I want **a dashboard card showing recent score anomalies**,
So that I can **see significant changes at a glance** without checking email/Slack.

**Acceptance Criteria:**

**Given** I am on the admin dashboard
**When** anomalies have been detected in the last 24 hours
**Then** I see an "Anomaly Alerts" card showing:
  - List of DAOs with significant changes
  - Old score → New score with delta
  - Color coding: red for drops, green for increases
  - Timestamp of detection

**Given** no anomalies in the last 24 hours
**When** I view the dashboard
**Then** the card shows "No significant changes detected"
**And** the card has a neutral/green status indicator

**Given** I click on an anomaly entry
**When** the detail expands
**Then** I can see: DAO name, snapshot space, full score history link (future)

**Technical Notes:**
- New component: `src/components/admin/anomaly-alerts.tsx`
- Fetch anomalies from job logs or dedicated table
- Place in Overview section or Status Overview panel
- Use shadcn/ui Alert component with appropriate variants

---

## Summary

| Epic | Stories | FRs Covered |
|------|---------|-------------|
| Epic 1: Dashboard Layout & Navigation | 3 stories (1.1, 1.2, 1.3) | FR1, FR2, FR6 |
| Epic 2: Operations Hub (Reports & Actions) | 3 stories (2.1, 2.2, 2.3) | FR3, FR4, FR7 |
| Epic 3: Content Management | 2 stories (3.1, 3.2) | FR5 |
| Epic 4: Daily GVS Monitoring & Historical Tracking | 5 stories (4.1, 4.2, 4.3, 4.4, 4.5) | FR8, FR9, FR10 |

**Total: 13 stories across 4 epics**

All 10 Functional Requirements are covered by the stories above.
