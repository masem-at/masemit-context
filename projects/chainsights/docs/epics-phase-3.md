# ChainSights Phase 3: Engagement Hub

**Status:** Planning
**Created:** 2026-02-04
**Source:** docs/_masemIT/requirements/ticket-engagement-hub.md

---

## Overview

Phase 3 focuses on building internal tools to scale community engagement across all 44 tracked DAOs. The Engagement Hub provides a central cockpit for tracking outreach, logging interactions, and (later) automatically surfacing engagement opportunities.

---

## Epic 19: Engagement Hub (Phase 1 + 2)

**Priority:** High
**Effort:** Medium
**Dependencies:** Existing `daos` table, Admin Auth (✅ exists)

### Business Context

ChainSights tracks 44 DAOs, but community engagement (forum posts, Discord, X replies, DMs) happens completely manually. There's no central view of:
- Which DAOs have we engaged with?
- What channels does each DAO have?
- When was our last interaction?
- What's the status of ongoing conversations?

### Scope: Phase 1 + 2 (MVP)

**Phase 1: DAO Community Links**
- Extend `daos` table with community link fields
- Admin UI to edit Twitter handle, Forum URL, Discord URL per DAO
- Engagement status tracking (not_started, active, warm, pending, blocked)
- Engagement notes field for free-text context

**Phase 2: Engagement Log**
- New `engagement_log` table for interaction history
- Timeline view per DAO showing all logged interactions
- Manual "Add Entry" form for logging forum posts, X replies, DMs
- Outcome tracking (no_response, replied, positive, negative, blocked)

### Out of Scope (Phase 3a/b/c - Future)

- Automated X/Twitter signal scanner (X API access: **beantragt**)
- Forum RSS monitoring
- Discord webhook integration
- AI signal evaluation

---

## Stories

### Story 19-1: Extend DAOs Table with Community Links

**As a** ChainSights admin
**I want** to store community links for each DAO
**So that** I can quickly access their Twitter, Forum, and Discord from one place

#### Acceptance Criteria
- [ ] Migration adds columns to `daos` table:
  - `twitter_handle` (VARCHAR 100)
  - `forum_url` (VARCHAR 500)
  - `discord_url` (VARCHAR 500)
  - `governance_forum_url` (VARCHAR 500, if different from forum)
  - `telegram_url` (VARCHAR 500)
  - `engagement_status` (VARCHAR 20, default 'not_started')
  - `engagement_notes` (TEXT)
  - `last_engagement_at` (TIMESTAMP)
- [ ] Drizzle schema updated with new fields
- [ ] Types exported for use in admin UI

#### Technical Notes
```sql
ALTER TABLE daos ADD COLUMN twitter_handle VARCHAR(100);
ALTER TABLE daos ADD COLUMN forum_url VARCHAR(500);
ALTER TABLE daos ADD COLUMN discord_url VARCHAR(500);
ALTER TABLE daos ADD COLUMN governance_forum_url VARCHAR(500);
ALTER TABLE daos ADD COLUMN telegram_url VARCHAR(500);
ALTER TABLE daos ADD COLUMN engagement_status VARCHAR(20) DEFAULT 'not_started';
ALTER TABLE daos ADD COLUMN engagement_notes TEXT;
ALTER TABLE daos ADD COLUMN last_engagement_at TIMESTAMP;
```

---

### Story 19-2: Admin DAO Edit - Community Links Section

**As a** ChainSights admin
**I want** to edit community links for a DAO in the admin panel
**So that** I can maintain accurate contact information

#### Acceptance Criteria
- [ ] DAO edit page shows "Community Links" section
- [ ] Editable fields: Twitter handle, Forum URL, Discord URL, Telegram URL
- [ ] Engagement status dropdown (not_started, active, warm, pending, blocked)
- [ ] Last engagement date picker
- [ ] Engagement notes textarea
- [ ] Save persists all fields to database
- [ ] Validation: URLs must be valid format, Twitter handle with/without @

#### UI Reference
```
── Community Links ──
X/Twitter:  [@LidoFinance          ]
Forum:      [https://research.lido.fi  ]
Discord:    [https://discord.gg/lido   ]
Telegram:   [                          ]

── Engagement Tracking ──
Status: [Active ▾]
Last Active: [2026-02-04]
Notes:
┌──────────────────────────────────────┐
│ BCV replied twice on governance     │
│ health post. Good contact.          │
└──────────────────────────────────────┘
```

---

### Story 19-3: Engagement Hub Overview Page

**As a** ChainSights admin
**I want** a dashboard showing all DAOs with their engagement status
**So that** I can see at a glance where we're active and where we need to engage

#### Acceptance Criteria
- [ ] New admin page at `/admin/engagement`
- [ ] Table showing all DAOs with columns:
  - DAO name
  - DGI score
  - Category
  - Community links (icons: 🐦 🗣️ 💬)
  - Engagement status (color-coded)
  - Last active date
- [ ] Sortable by: DGI score, status, last active
- [ ] Filterable by: category, engagement status, has forum, has twitter
- [ ] Quick stats header: total DAOs, active engagements, pending responses
- [ ] Click row → opens DAO edit page
- [ ] Color coding:
  - 🔴 Red = needs attention (no activity > 7 days but marked active)
  - 🟡 Yellow = warm/pending
  - 🟢 Green = active
  - ⚪ Gray = not started

#### Technical Notes
- Server component with data fetching
- Client component for sorting/filtering
- Reuse existing admin layout and styles

---

### Story 19-4: Create Engagement Log Table

**As a** ChainSights admin
**I want** to log my interactions with DAOs
**So that** I can track conversation history and outcomes

#### Acceptance Criteria
- [ ] Migration creates `engagement_log` table:
  - `id` (UUID, primary key)
  - `dao_id` (UUID, FK to daos)
  - `type` (VARCHAR 20): forum_post, forum_reply, x_reply, x_post, discord, dm, note
  - `platform` (VARCHAR 20): x, forum, discord, telegram, other
  - `content` (TEXT, optional)
  - `url` (VARCHAR 500, optional)
  - `contact_name` (VARCHAR 100, optional)
  - `outcome` (VARCHAR 20): no_response, replied, positive, negative, blocked
  - `created_at` (TIMESTAMP)
- [ ] Drizzle schema with proper relations to daos table
- [ ] Index on dao_id and created_at for fast queries

---

### Story 19-5: Engagement Timeline per DAO

**As a** ChainSights admin
**I want** to see a timeline of all interactions with a specific DAO
**So that** I can review conversation history before engaging

#### Acceptance Criteria
- [ ] New page at `/admin/engagement/[daoId]`
- [ ] Shows DAO header with name, DGI score, community links
- [ ] Timeline of engagement log entries, newest first
- [ ] Each entry shows:
  - Date
  - Type icon (🗣️ forum, 🐦 X, 💬 Discord, ✉️ DM)
  - Content preview (truncated)
  - Contact name if present
  - Outcome badge
  - Link to original post/reply
- [ ] "Add Entry" button opens modal form
- [ ] Empty state with CTA to add first entry

---

### Story 19-6: Add Engagement Log Entry Form

**As a** ChainSights admin
**I want** to manually log an interaction
**So that** I can keep track of my outreach activities

#### Acceptance Criteria
- [ ] Modal form accessible from timeline page
- [ ] Fields:
  - Type (dropdown): Forum Post, Forum Reply, X Post, X Reply, Discord, DM, Note
  - Platform (auto-filled based on type, editable)
  - Content (textarea, optional)
  - URL (input, optional, validated)
  - Contact name (input, optional)
  - Outcome (dropdown): Pending, No Response, Replied, Positive, Negative, Blocked
  - Date (date picker, defaults to today)
- [ ] Save creates entry and updates DAO's `last_engagement_at`
- [ ] Success toast and modal closes
- [ ] New entry appears in timeline immediately

---

### Story 19-7: Engagement Hub Navigation & Analytics

**As a** ChainSights admin
**I want** easy access to the Engagement Hub
**So that** I can quickly check and update engagement status

#### Acceptance Criteria
- [ ] "Engagement" link in admin sidebar navigation
- [ ] Badge showing count of DAOs needing attention (status=active but last_engagement > 7 days)
- [ ] Analytics events:
  - `admin_engagement_view` - Hub page viewed
  - `admin_engagement_log_add` - Entry added
  - `admin_dao_engagement_edit` - DAO engagement fields edited
- [ ] Breadcrumb navigation: Admin > Engagement > [DAO Name]

---

## Technical Architecture

### Database Schema

```
daos (existing, extended)
├── twitter_handle
├── forum_url
├── discord_url
├── governance_forum_url
├── telegram_url
├── engagement_status
├── engagement_notes
└── last_engagement_at

engagement_log (new)
├── id (PK)
├── dao_id (FK → daos)
├── type
├── platform
├── content
├── url
├── contact_name
├── outcome
└── created_at
```

### File Structure

```
src/app/admin/engagement/
├── page.tsx                    # Hub overview
├── [daoId]/
│   └── page.tsx               # DAO timeline
└── components/
    ├── engagement-table.tsx   # Sortable/filterable table
    ├── engagement-filters.tsx # Filter controls
    ├── log-entry-modal.tsx    # Add entry form
    └── timeline-entry.tsx     # Single timeline item

src/lib/db/schema.ts           # Extended with engagement fields
drizzle/0017_*.sql             # Migration for daos columns
drizzle/0018_*.sql             # Migration for engagement_log table
```

---

## Future: Phase 3a/b/c (Signal Scanner)

**Not in current scope, but planned:**

- **Phase 3a:** X/Twitter Signal Scanner
  - Search API queries for governance discussions
  - 3x daily cron job
  - Requires X API access (Status: **beantragt**)

- **Phase 3b:** Forum RSS Scanner
  - Monitor Discourse forums via RSS
  - Keyword matching for governance topics

- **Phase 3c:** Discord Monitoring
  - Requires bot setup per server
  - Lowest priority

- **Phase 4:** AI Signal Evaluation
  - Prioritize signals by relevance
  - Generate hook suggestions

---

## Dependencies

| Dependency | Status |
|------------|--------|
| Admin authentication | ✅ Exists |
| `daos` table | ✅ Exists |
| Admin layout/styles | ✅ Exists |
| X API access | 🟡 Beantragt |

---

## Acceptance Criteria Summary

- [ ] `daos` table extended with community link fields
- [ ] Admin DAO edit page shows community links section
- [ ] Admin DAO edit page shows engagement tracking (status, notes, last active)
- [ ] `/admin/engagement` overview page with sortable/filterable table
- [ ] Color-coded engagement status per DAO
- [ ] `engagement_log` table exists
- [ ] Engagement timeline viewable per DAO
- [ ] Manual "Add Entry" form for logging interactions
- [ ] Quick stats: total engaged, active, pending
- [ ] Navigation link with attention badge
