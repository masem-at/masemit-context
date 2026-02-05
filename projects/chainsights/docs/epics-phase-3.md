# ChainSights Phase 3: Engagement Hub

**Status:** In Progress
**Created:** 2026-02-04
**Updated:** 2026-02-05
**Source:** docs/_masemIT/requirements/ticket-engagement-hub.md

---

## Overview

Phase 3 focuses on building internal tools to scale community engagement across all 44 tracked DAOs. The Engagement Hub provides a central cockpit for tracking outreach, logging interactions, and (later) automatically surfacing engagement opportunities.

---

## Epic 19: Engagement Hub (Phase 1 + 2) ✅ DONE

**Priority:** High
**Effort:** Medium
**Status:** ✅ Complete (2026-02-05)
**Dependencies:** Existing `daos` table, Admin Auth (✅ exists)

### Summary

Implemented the manual Engagement Hub with:
- DAO community links (Twitter, Forum, Discord, Telegram)
- Engagement status tracking per DAO
- Engagement log for interaction history
- Timeline view per DAO
- Admin UI at `/admin/engagement`

---

## Epic 20: Engagement Report (PDF)

**Priority:** High
**Effort:** Medium
**Dependencies:** Epic 19 (Engagement Hub) ✅
**Trigger:** Manual button in Admin

### Business Context

In Web3, building relationships is critical. One DM to a DAO leader can result in 45+ referrals in an hour (Radiant Capital example). Mario needs a weekly overview to:
- Track which DAOs are engaged vs. not
- See what activities happened
- Plan the upcoming week's outreach
- Demonstrate ROI of engagement efforts

### Report Structure

#### 1. Header / Summary
- Report period (e.g., "KW 5/2026" or "January 2026")
- Total DAOs engaged vs. Total DAOs tracked (e.g., 4/46)
- Total activities (Posts, Replies, DMs)
- New contacts made

#### 2. Highlights
- Top successes (e.g., "Radiant Capital shared our DGI organically — 45 referrals in one day")
- New connections with names and roles
- Traffic impact from community channels (deferred - pending analytics team discussion)

#### 3. Per DAO Breakdown (active DAOs only)
- DAO Name, DGI Score, Engagement Status
- Activities timeline (compact)
- Key contacts with their roles
- Next steps / Open items

#### 4. Engagement Metrics
- Posts/Replies by Platform (Forum, Discord, X, DM)
- Response rate: How many outreach attempts got replies
- Status distribution: Active / Warm / Pending / Not Started (Donut chart)

#### 5. Pipeline / Next Actions
- Open DMs without response
- Planned posts
- DAOs to approach next

#### 6. Traffic Attribution ✅ API READY
- Referrals from community channels (from MMS Analytics API)
- Which engagement channel drives actual visitors
- **API:** `GET /v1/analytics/referrers` — [docs/integrations/mms-analytics-api.md](integrations/mms-analytics-api.md)

### Acceptance Criteria

- [ ] "Generate Report" button in `/admin/engagement`
- [ ] Date range selector (week picker, defaults to last week)
- [ ] PDF generation using existing PDF infrastructure
- [x] Report includes all sections including Traffic Attribution
- [ ] Professional styling consistent with Governance Reports
- [ ] Download triggers immediately (no email)

---

## Stories for Epic 20

### Story 20-1: Create Engagement Report Data Aggregation

**As a** ChainSights admin
**I want** to aggregate engagement data for a date range
**So that** I can generate meaningful reports

#### Acceptance Criteria
- [ ] Query function `getEngagementReportData(startDate, endDate)`
- [ ] Returns:
  - Summary stats (DAOs engaged, total activities, new contacts)
  - Per-DAO breakdown with activities and contacts
  - Platform distribution (forum, x, discord, dm counts)
  - Response rate calculation
  - Status distribution counts
- [ ] Efficient queries using existing indexes
- [ ] Handles empty data gracefully

---

### Story 20-2: Build Engagement Report PDF Template

**As a** ChainSights admin
**I want** a professional PDF layout for the engagement report
**So that** I can review and share it

#### Acceptance Criteria
- [ ] PDF template using @react-pdf/renderer (existing infrastructure)
- [ ] ChainSights branding (logo, colors, fonts)
- [ ] Sections:
  - Header with date range and summary stats
  - Highlights section (manual input or auto-detected)
  - Per-DAO cards with timeline and contacts
  - Metrics section with charts
  - Next Actions list
- [ ] Responsive layout for varying content lengths
- [ ] Consistent with existing Governance Report styling

---

### Story 20-3: Add Report Generation UI to Admin

**As a** ChainSights admin
**I want** a button to generate the engagement report
**So that** I can get my weekly report on demand

#### Acceptance Criteria
- [ ] "Generate Report" button on `/admin/engagement` page
- [ ] Week picker component (defaults to previous week)
- [ ] Loading state during PDF generation
- [ ] PDF downloads directly to browser
- [ ] Error handling with user-friendly message
- [ ] Analytics event: `admin_engagement_report_generated`

---

### Story 20-4: Add Highlights Input (Optional Enhancement)

**As a** ChainSights admin
**I want** to add custom highlights to the report
**So that** I can include context that isn't in the database

#### Acceptance Criteria
- [ ] Optional text input for highlights before generation
- [ ] Supports multiple highlight entries
- [ ] Highlights appear in the PDF Highlights section
- [ ] Can generate without highlights (section shows auto-detected only)

---

### Story 20-5: Integrate MMS Analytics for Traffic Attribution

**As a** ChainSights admin
**I want** to see which community channels drive traffic
**So that** I can measure the ROI of engagement efforts

#### API Reference
- **Docs:** [docs/integrations/mms-analytics-api.md](integrations/mms-analytics-api.md)
- **Endpoint:** `GET /v1/analytics/referrers?project=chainsights`
- **Env:** `MMS_API_URL`, `MMS_API_KEY` (already configured)

#### Acceptance Criteria
- [x] Create `src/lib/mms/analytics.ts` client module
- [x] `getReferrers(from, to, limit)` function with proper error handling
- [x] Type definitions for API response (`ReferrersResponse`)
- [x] `getCommunityTraffic()` helper for filtered community channels
- [x] `formatChannelName()` for user-friendly display names
- [x] Graceful fallback if API unavailable (returns null, not error)
- [x] Integrate into `getEngagementReportData()` (Story 20-1)
- [x] PDF section shows: domain, visits, pageviews, engagement ratio

#### Example Output in PDF
```
Traffic from Community Channels (KW 5)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Discord     45 visits   98 pv   2.2x engagement
X/Twitter   23 visits   31 pv   1.3x engagement
Forum       12 visits   34 pv   2.8x engagement
```

---

## Deferred: Phase 3a/b/c (Signal Scanner)

**Status:** Blocked — X API Access beantragt

- **Phase 3a:** X/Twitter Signal Scanner
- **Phase 3b:** Forum RSS Scanner
- **Phase 3c:** Discord Monitoring
- **Phase 4:** AI Signal Evaluation

---

## Traffic Attribution Integration

**Status:** ✅ Complete (Story 20-5)
**API Docs:** [docs/integrations/mms-analytics-api.md](integrations/mms-analytics-api.md)

**Env Variables (configured):**
- `MMS_API_URL=https://api.masem.at`
- `MMS_API_KEY=mms_chainsights_xxxxx`

**Endpoint:** `GET /v1/analytics/referrers`
- Returns: domain, visits, pageviews, visitors per referrer
- Parameters: project=chainsights, from, to, limit

**Use in Report:**
- Which engagement channels drive traffic
- Visits + Pageviews = engagement quality metric
- ROI per platform/DAO

---

## Backlog Items

### Trust Builder Feature

**Source:** Party Mode session 2026-02-05
**Status:** Idea - needs definition

Concept for building trust signals on the platform. Details TBD.

---

## Dependencies

| Dependency | Status |
|------------|--------|
| Admin authentication | ✅ Exists |
| Engagement Hub (Epic 19) | ✅ Done |
| PDF generation infrastructure | ✅ Exists (Governance Reports) |
| X API access | 🟡 Beantragt |
| Analytics API | ⏸️ Pending discussion |
