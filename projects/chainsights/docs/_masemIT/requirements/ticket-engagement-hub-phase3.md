# DEV TICKET: Engagement Hub (Admin) + Governance Signal Scanner

**Priority:** Medium-High
**Effort:** Medium-Large (phased)
**Dependencies:** Existing `daos` table, Admin Auth (✅ exists)

---

## Context

ChainSights trackt 44 DAOs, aber die Community-Engagement-Arbeit (Forum-Posts, Discord, X/Twitter Replies, DMs) passiert komplett manuell. Mario scrollt durch Feeds, sucht nach Hooks, und merkt sich mental wo er aktiv ist und wo nicht.

Was fehlt: Ein zentrales Cockpit das pro DAO alle relevanten Community-Kanäle zeigt, Engagement-Aktivitäten trackt, und automatisch Signale findet die einen Reply wert sind.

---

## Vision

```
┌─────────────────────────────────────────────────────────────────────┐
│  ENGAGEMENT HUB                                          [Admin]   │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  🔴 3 Hooks found    🟡 2 Warm leads    📊 Last scan: 14:00 UTC    │
│                                                                     │
│  [Filter: All | 🔴 Hooks | Category ▾ | Has Forum ▾]              │
│                                                                     │
├──────────┬──────┬──────┬───────┬────────┬────────┬─────────────────┤
│ DAO      │ DGI  │ Cat  │ Links │ Status │ Last   │ Signals         │
│          │      │      │       │        │ Active │                 │
├──────────┼──────┼──────┼───────┼────────┼────────┼─────────────────┤
│ 🔴 Lido  │ 7.3  │ DeFi │ 🐦🗣️💬 │ Active │ 2d ago │ BCV replied!   │
│ 🔴 ENS   │ 7.1  │ Infra│ 🐦🗣️💬 │ Pending│ 5d ago │ New delegation  │
│          │      │      │       │        │        │ proposal        │
│ 🔴 Rock. │ 6.8  │ DeFi │ 🐦🗣️  │ New    │ today  │ Inactive deleg. │
│          │      │      │       │        │        │ discussion      │
│ 🟡 Giveth│ 7.7  │ PG   │ 🗣️💬  │ DM sent│ today  │ Lauren - await  │
│ ⚪ Aave  │ 6.4  │ DeFi │ 🐦🗣️💬 │ —      │ never  │ —               │
│ ⚪ Uniswp│ 6.8  │ DeFi │ 🐦🗣️💬 │ —      │ never  │ —               │
│ ...      │      │      │       │        │        │                 │
└──────────┴──────┴──────┴───────┴────────┴────────┴─────────────────┘
```

**Legend:**
- 🔴 Hook found — actionable engagement opportunity
- 🟡 Warm lead — awaiting response or in conversation
- ⚪ No activity — not yet engaged
- 🐦 = X/Twitter handle | 🗣️ = Forum link | 💬 = Discord link

---

## Phase 1: DAO Community Links (Admin Page)

### Extend `daos` table

Add community link fields to existing `daos` table:

```sql
ALTER TABLE daos ADD COLUMN twitter_handle VARCHAR(100);    -- e.g. "@LidoFinance"
ALTER TABLE daos ADD COLUMN forum_url VARCHAR(500);          -- e.g. "https://research.lido.fi"
ALTER TABLE daos ADD COLUMN discord_url VARCHAR(500);        -- e.g. "https://discord.gg/lido"
ALTER TABLE daos ADD COLUMN governance_forum_url VARCHAR(500);-- if different from forum_url
ALTER TABLE daos ADD COLUMN telegram_url VARCHAR(500);
ALTER TABLE daos ADD COLUMN engagement_status VARCHAR(20)    -- 'not_started' | 'active' | 'warm' | 'pending' | 'blocked'
  DEFAULT 'not_started';
ALTER TABLE daos ADD COLUMN engagement_notes TEXT;           -- free text notes
ALTER TABLE daos ADD COLUMN last_engagement_at TIMESTAMP;    -- when was last interaction
```

### Admin UI: DAO Edit Page

Extend existing admin DAO management with community links section:

```
┌─────────────────────────────────────────────┐
│  Edit DAO: Lido                             │
├─────────────────────────────────────────────┤
│                                             │
│  Snapshot Space: lido-snapshot.eth          │
│  Category: [DeFi ▾]                         │
│  DGI Score: 7.3 (auto)                      │
│                                             │
│  ── Community Links ──                      │
│                                             │
│  X/Twitter:  [@LidoFinance          ]       │
│  Forum:      [https://research.lido.fi  ]   │
│  Discord:    [https://discord.gg/lido   ]   │
│  Telegram:   [                          ]   │
│                                             │
│  ── Engagement Tracking ──                  │
│                                             │
│  Status: [Active ▾]                         │
│  Last Active: 2026-02-04                    │
│  Notes:                                     │
│  ┌──────────────────────────────────────┐   │
│  │ BCV replied twice on governance     │   │
│  │ health post. Good contact. Follow   │   │
│  │ up on delegation incentives thread. │   │
│  └──────────────────────────────────────┘   │
│                                             │
│  [Save]                                     │
│                                             │
└─────────────────────────────────────────────┘
```

### Admin UI: Engagement Hub Overview

New admin page `/admin/engagement` that shows the overview table from the Vision section above.

**Features:**
- Sortable by DGI score, engagement status, last active date
- Filterable by category, engagement status, "has forum", "has twitter"
- Color-coded rows based on engagement status
- Click row → opens DAO edit page
- Quick stats header: total DAOs, active engagements, hooks found

---

## Phase 2: Engagement Log

### New `engagement_log` table

```sql
CREATE TABLE engagement_log (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  dao_id UUID REFERENCES daos(id),
  type VARCHAR(20) NOT NULL,       -- 'forum_post' | 'forum_reply' | 'x_reply' | 'x_post' | 'discord' | 'dm' | 'note'
  platform VARCHAR(20) NOT NULL,   -- 'x' | 'forum' | 'discord' | 'telegram' | 'other'
  content TEXT,                     -- what was posted/sent (optional)
  url VARCHAR(500),                 -- link to the post/reply
  contact_name VARCHAR(100),        -- who we interacted with (e.g. "BCV", "Lauren Luz")
  outcome VARCHAR(20),              -- 'no_response' | 'replied' | 'positive' | 'negative' | 'blocked'
  created_at TIMESTAMP DEFAULT NOW()
);
```

### Engagement Timeline per DAO

```
┌─────────────────────────────────────────────────────────┐
│  Lido — Engagement History                              │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  📅 Feb 4, 2026                                        │
│  └─ 🗣️ Forum Reply — replied to BCV on delegation     │
│     Contact: BCV (Grants Committee)                     │
│     Outcome: ✅ Replied                                 │
│     Link: research.lido.fi/t/...                        │
│                                                         │
│  📅 Feb 3, 2026                                        │
│  └─ 🗣️ Forum Post — "Governance Health Analysis"      │
│     Outcome: ✅ 2 Replies (BCV)                        │
│     Link: research.lido.fi/t/...                        │
│                                                         │
│  📅 Jan 28, 2026                                       │
│  └─ 🐦 X Reply — replied to @LidoFinance thread       │
│     Outcome: ⚪ No response                            │
│                                                         │
│  [+ Add Entry]                                          │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

---

## Phase 3: Governance Signal Scanner (Automated)

### Concept

Automated scanning of DAO community channels for engagement opportunities ("hooks"). Runs 3x daily via cron, surfaces results in the Engagement Hub.

### New `signals` table

```sql
CREATE TABLE signals (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  dao_id UUID REFERENCES daos(id),
  source VARCHAR(20) NOT NULL,      -- 'x_search' | 'forum_rss' | 'discord_webhook'
  signal_type VARCHAR(20) NOT NULL, -- 'governance_discussion' | 'delegation' | 'frustration' | 'question' | 'mention'
  priority VARCHAR(10) NOT NULL,    -- 'high' | 'medium' | 'low'
  title VARCHAR(500),
  content TEXT,
  url VARCHAR(500),
  author VARCHAR(100),
  detected_at TIMESTAMP DEFAULT NOW(),
  status VARCHAR(20) DEFAULT 'new', -- 'new' | 'reviewed' | 'acted' | 'dismissed'
  acted_at TIMESTAMP,
  hook_suggestion TEXT               -- AI-generated hook direction
);
```

### Signal Sources (Priority Order)

**1. X/Twitter Search API (Phase 3a)**
- Search queries per DAO: `"@{handle} governance"`, `"@{handle} delegate"`, `"@{handle} voting"`
- Global governance queries: `"DAO governance" frustrated`, `"voting power" concentrated`
- Filter: last 7 days, min 5 likes, exclude bots/shills
- 3x daily scan (08:00, 14:00, 20:00 UTC)

**2. Forum RSS/API (Phase 3b)**
- Discourse forums expose RSS: `{forum_url}/latest.rss`
- Monitor for new topics in governance categories
- Keyword matching: "delegation", "voting power", "participation", "decentralization"

**3. Discord Webhooks (Phase 3c — Future)**
- Requires bot in each Discord server
- Monitor governance channels for relevant discussions
- Most complex, lowest priority

### AI Signal Evaluation

Each detected signal gets evaluated by AI (similar to X Hook Scout):

```typescript
const evaluation = {
  priority: "high",     // 🔴 Sofort / 🟡 Gut / 🟢 Soft
  relevance: 0.85,      // 0-1 score
  hookDirection: "Confirm their frustration about inactive delegates. 
                  Mention that concentration data shows 65% controlled by top 5.",
  triggerTopics: ["delegate_concentration", "inactive_delegation"],
  shouldEngage: true
};
```

### Signal Dashboard in Engagement Hub

```
┌─────────────────────────────────────────────────────────────────┐
│  🔔 SIGNALS                                    [Scan Now]      │
│  Last scan: 14:00 UTC | Next: 20:00 UTC                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  🔴 HIGH — Rocket Pool (@Rocket_Pool) — 2h ago                │
│  "Is your governance power effective? If you've delegated..."   │
│  Hook: Inactive delegation → our concentration data             │
│  [Reply Draft] [Dismiss] [View on X]                           │
│                                                                 │
│  🔴 HIGH — ENS Forum — 5h ago                                  │
│  New topic: "[Temp Check] Delegation Incentives Program"        │
│  Hook: Direct DGI data for ENS delegation metrics               │
│  [Reply Draft] [Dismiss] [View on Forum]                       │
│                                                                 │
│  🟡 MEDIUM — Lido Forum — 1d ago                               │
│  BCV posted in delegation thread (new reply)                    │
│  Hook: Continue existing conversation with fresh data           │
│  [Reply Draft] [Dismiss] [View on Forum]                       │
│                                                                 │
│  🟢 LOW — Gitcoin (@GitcoinDAO) — 3d ago                      │
│  "Governance participation hit all-time low this quarter"       │
│  Hook: Public Goods category averaging 3.3/10 in DGI           │
│  [Reply Draft] [Dismiss] [View on X]                           │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## Implementation Phases

| Phase | What | Effort | Dependencies |
|-------|------|--------|--------------|
| **Phase 1** | DAO community links + Engagement Hub overview | Small | daos table, admin auth |
| **Phase 2** | Engagement log + timeline per DAO | Small | Phase 1 |
| **Phase 3a** | X/Twitter signal scanner (cron) | Medium | Phase 1, X API access |
| **Phase 3b** | Forum RSS scanner (cron) | Small | Phase 1 |
| **Phase 3c** | Discord monitoring | Large | Discord bot setup per server |
| **Phase 4** | AI evaluation of signals | Medium | Phase 3a/3b, AI API |

**Recommended start:** Phase 1 + 2 together. This gives you the manual Engagement Hub immediately, and you can populate it while using the tool. Phase 3 automates what you're doing manually today.

---

## Acceptance Criteria (Phase 1 + 2)

- [ ] `daos` table extended with community link fields
- [ ] Admin edit page shows community links section
- [ ] Admin edit page shows engagement tracking (status, notes, last active)
- [ ] `/admin/engagement` overview page with sortable/filterable table
- [ ] Color-coded engagement status per DAO
- [ ] Engagement log table exists
- [ ] Engagement timeline viewable per DAO
- [ ] Manual "Add Entry" form for logging interactions
- [ ] Quick stats: total engaged, hooks found, awaiting response

## Acceptance Criteria (Phase 3)

- [ ] Cron job runs 3x daily
- [ ] X/Twitter search results stored as signals
- [ ] Forum RSS monitored for governance topics
- [ ] Signals displayed in Engagement Hub with priority
- [ ] AI evaluation generates hook suggestions
- [ ] "Dismiss" / "Acted" workflow for signal management
- [ ] Signal count badge visible in admin nav

---

## Files to Create/Touch

### Phase 1 + 2
1. DB migration — extend `daos` table + create `engagement_log` table
2. `src/app/admin/engagement/page.tsx` — Engagement Hub overview
3. Admin DAO edit page — add community links + engagement section
4. `src/app/admin/engagement/[daoId]/page.tsx` — DAO engagement timeline
5. API routes for CRUD on engagement_log

### Phase 3
6. DB migration — create `signals` table
7. `src/lib/signals/x-scanner.ts` — X/Twitter search scanner
8. `src/lib/signals/forum-scanner.ts` — Forum RSS scanner
9. `src/lib/signals/evaluate.ts` — AI signal evaluation
10. Cron job configuration (QStash or Vercel Cron)
11. Signal management UI in Engagement Hub
