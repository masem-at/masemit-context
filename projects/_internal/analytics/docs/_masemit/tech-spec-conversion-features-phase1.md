# Tech Spec: Analytics Conversion Features – Phase 1

**Project:** masemIT Analytics
**Version:** 1.0
**Author:** Winston (Architect), Mary (Analyst), Sally (UX)
**Date:** 2026-01-28
**Status:** Draft

---

## 1. Overview

### 1.1 Scope

Phase 1 implements four generic conversion analytics features for all masemIT projects:

| Feature | Priority | Complexity |
|---------|----------|------------|
| FR-1: Funnel Visualization | Highest | Medium |
| FR-2: CTA Performance Table | High | Low |
| FR-3: Time to Conversion | Medium | Medium |
| FR-4: Returning vs. New Visitors | Medium | Low |

### 1.2 Prerequisites

- **FR-0: Stripe Webhook Integration** must be completed in ChainSights and tellingCube
- See `handoff-stripe-webhook-analytics.md` for implementation details

### 1.3 Architecture Decision

**Tab-based Navigation** using URL query parameters:

```
/dashboard/[project]?tab=overview    ← Existing (default)
/dashboard/[project]?tab=funnel      ← FR-1
/dashboard/[project]?tab=cta         ← FR-2
/dashboard/[project]?tab=conversions ← FR-3 + FR-4
```

**Rationale:**
- URL state = shareable, bookmarkable
- Server Components can read `searchParams` directly
- No client state management needed
- Consistent with existing `?range=` pattern

---

## 2. Database Schema Updates

### 2.1 Events Table Extension

```sql
-- Add first_visit tracking to events table
-- This allows us to identify returning visitors without a separate sessions table

ALTER TABLE events
ADD COLUMN is_first_visit BOOLEAN DEFAULT TRUE;

-- Index for efficient returning visitor queries
CREATE INDEX idx_events_device_first_visit
ON events (project_id, device_id, is_first_visit);

-- Index for time-to-conversion queries
CREATE INDEX idx_events_device_timestamp
ON events (project_id, device_id, timestamp);
```

### 2.2 Update Logic

When inserting events via `/api/events`, check if `device_id` has been seen before:

```typescript
// In app/api/events/route.ts - enhanced insert logic
const isFirstVisit = await sql`
  SELECT COUNT(*) = 0 as is_first
  FROM events
  WHERE project_id = ${projectId}
    AND device_id = ${deviceId}
  LIMIT 1
`;

await sql`
  INSERT INTO events (..., is_first_visit)
  VALUES (..., ${isFirstVisit})
`;
```

### 2.3 Migration Script

```sql
-- One-time backfill for existing data
-- Run manually after schema update

WITH first_visits AS (
  SELECT DISTINCT ON (project_id, device_id)
    id, project_id, device_id
  FROM events
  ORDER BY project_id, device_id, timestamp ASC
)
UPDATE events e
SET is_first_visit = (e.id = fv.id)
FROM first_visits fv
WHERE e.project_id = fv.project_id
  AND e.device_id = fv.device_id;
```

---

## 3. Funnel Configuration

### 3.1 Configuration Structure

```typescript
// lib/config/funnels.ts

export interface FunnelStep {
  id: string;
  label: string;
  event: string;
  filter?: {
    path?: string;
    event_data?: Record<string, unknown>;
  };
}

export interface FunnelConfig {
  projectId: string;
  steps: FunnelStep[];
}

export const FUNNEL_CONFIGS: Record<string, FunnelConfig> = {
  chainsights: {
    projectId: 'chainsights',
    steps: [
      { id: 'landing', label: 'Landing', event: 'pageView', filter: { path: '/' } },
      { id: 'explore', label: 'Explore', event: 'pageView', filter: { path: '/rankings' } },
      { id: 'tier_view', label: 'Tier Modal', event: 'tier_view' },
      { id: 'quick_check', label: 'Quick Check', event: 'quick_check_complete' },
      { id: 'checkout', label: 'Checkout', event: 'checkout_start' },
      { id: 'paid', label: 'Paid', event: 'checkout_complete' },
    ],
  },

  'telling-cube': {
    projectId: 'telling-cube',
    steps: [
      { id: 'landing', label: 'Landing', event: 'pageView', filter: { path: '/' } },
      { id: 'template', label: 'Template View', event: 'template_view' },
      { id: 'generate', label: 'Generate', event: 'generate_start' },
      { id: 'download', label: 'Download', event: 'scenario_download' },
      { id: 'checkout', label: 'Checkout', event: 'checkout_start' },
      { id: 'paid', label: 'Paid', event: 'checkout_complete' },
    ],
  },

  'hoki-help': {
    projectId: 'hoki-help',
    steps: [
      { id: 'landing', label: 'Landing', event: 'pageView', filter: { path: '/' } },
      { id: 'cta_click', label: 'Donate Click', event: 'cta_click' },
      { id: 'amount', label: 'Amount Select', event: 'amount_select' },
      { id: 'donated', label: 'Donated', event: 'donate_complete' },
    ],
  },

  'masemit-site': {
    projectId: 'masemit-site',
    steps: [
      { id: 'landing', label: 'Landing', event: 'pageView', filter: { path: '/' } },
      { id: 'services', label: 'Services', event: 'pageView', filter: { path: '/services' } },
      { id: 'contact', label: 'Contact Click', event: 'cta_click' },
    ],
  },
};
```

### 3.2 Funnel Query

```typescript
// lib/queries/funnel.ts

export async function getFunnelData(
  projectId: string,
  dateRange: { start: Date; end: Date }
): Promise<FunnelData> {
  const config = FUNNEL_CONFIGS[projectId];
  if (!config) throw new Error(`No funnel config for ${projectId}`);

  const stepCounts = await Promise.all(
    config.steps.map(async (step) => {
      const count = await sql`
        SELECT COUNT(DISTINCT device_id) as count
        FROM events
        WHERE project_id = ${projectId}
          AND timestamp >= ${dateRange.start}
          AND timestamp < ${dateRange.end}
          AND (
            ${step.event === 'pageView'
              ? sql`type = 'pageView' AND path = ${step.filter?.path}`
              : sql`type = ${step.event}`
            }
          )
      `;
      return {
        ...step,
        count: Number(count[0].count),
      };
    })
  );

  // Calculate percentages and drop-offs
  return {
    projectId,
    dateRange,
    steps: stepCounts.map((step, i) => ({
      ...step,
      percentage: i === 0 ? 100 : Math.round((step.count / stepCounts[0].count) * 100),
      dropOff: i === 0 ? 0 : Math.round(((stepCounts[i-1].count - step.count) / stepCounts[i-1].count) * 100),
    })),
  };
}
```

---

## 4. Component Architecture

### 4.1 File Structure

```
app/dashboard/[project]/
├── page.tsx                      # Add tab routing logic
├── components/
│   ├── TabNavigation.tsx         # Tab header component
│   ├── OverviewTab.tsx           # Extracted from current page.tsx
│   ├── FunnelTab.tsx             # FR-1
│   ├── CtaTab.tsx                # FR-2
│   └── ConversionsTab.tsx        # FR-3 + FR-4
└── lib/
    └── (queries stay in /lib)

lib/
├── config/
│   └── funnels.ts                # Funnel configurations
└── queries/
    ├── funnel.ts                 # Funnel queries
    ├── cta.ts                    # CTA performance queries
    └── conversions.ts            # Time to conversion queries
```

### 4.2 Tab Routing (page.tsx)

```typescript
// app/dashboard/[project]/page.tsx

import { TabNavigation } from './components/TabNavigation';
import { OverviewTab } from './components/OverviewTab';
import { FunnelTab } from './components/FunnelTab';
import { CtaTab } from './components/CtaTab';
import { ConversionsTab } from './components/ConversionsTab';

interface PageProps {
  params: Promise<{ project: string }>;
  searchParams: Promise<{ tab?: string; range?: string }>;
}

export default async function ProjectDashboard({ params, searchParams }: PageProps) {
  const { project } = await params;
  const { tab = 'overview', range = '7' } = await searchParams;

  const tabs = [
    { id: 'overview', label: 'Overview' },
    { id: 'funnel', label: 'Funnel' },
    { id: 'cta', label: 'CTA Performance' },
    { id: 'conversions', label: 'Conversions' },
  ];

  return (
    <div className="min-h-screen bg-[#0a0a0f] text-white">
      <Header project={project} />

      <main className="max-w-7xl mx-auto px-6 py-8">
        <TabNavigation tabs={tabs} currentTab={tab} project={project} range={range} />

        {tab === 'overview' && <OverviewTab project={project} range={range} />}
        {tab === 'funnel' && <FunnelTab project={project} range={range} />}
        {tab === 'cta' && <CtaTab project={project} range={range} />}
        {tab === 'conversions' && <ConversionsTab project={project} range={range} />}
      </main>
    </div>
  );
}
```

### 4.3 Tab Navigation Component

```typescript
// app/dashboard/[project]/components/TabNavigation.tsx

'use client';

import Link from 'next/link';

interface Tab {
  id: string;
  label: string;
}

interface TabNavigationProps {
  tabs: Tab[];
  currentTab: string;
  project: string;
  range: string;
}

export function TabNavigation({ tabs, currentTab, project, range }: TabNavigationProps) {
  return (
    <nav className="flex gap-1 mb-8 border-b border-white/10 pb-4">
      {tabs.map((tab) => (
        <Link
          key={tab.id}
          href={`/dashboard/${project}?tab=${tab.id}&range=${range}`}
          className={`px-4 py-2 rounded-lg text-sm transition-colors ${
            currentTab === tab.id
              ? 'bg-white text-black font-medium'
              : 'text-slate-400 hover:text-white hover:bg-white/10'
          }`}
        >
          {tab.label}
        </Link>
      ))}
    </nav>
  );
}
```

---

## 5. UI Specifications (Dark Mode)

### 5.1 Funnel Visualization (FR-1)

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ bg-white/5 rounded-2xl border border-white/10 p-6                           │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  Conversion Funnel                                        [Last 7 days ▼]  │
│  text-lg font-semibold text-white                                          │
│                                                                             │
│  ┌────────────────────────────────────────────────────────────────────┐    │
│  │                                                                    │    │
│  │   Landing      Explore      Tier Modal    Quick Check    Paid     │    │
│  │   ████████████ ████████     ████          ██             █        │    │
│  │   bg-cyan-500  opacity based on percentage                        │    │
│  │                                                                    │    │
│  │   450          210          45            12             3        │    │
│  │   100%         47%          10%           2.7%           0.7%     │    │
│  │                                                                    │    │
│  │                ↓ -53%       ↓ -79%        ↓ -73%         ↓ -75%   │    │
│  │                text-rose-400 for negative                         │    │
│  │                                                                    │    │
│  └────────────────────────────────────────────────────────────────────┘    │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

**Colors:**
- Funnel bars: `bg-cyan-500` with opacity scaling (100% → 10%)
- Drop-off indicators: `text-rose-400`
- Labels: `text-slate-400` (secondary), `text-white` (primary numbers)

### 5.2 CTA Performance Table (FR-2)

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ CTA Performance                                           [Sort: CTR ▼]    │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  CTA Name                    │ Views │ Clicks │ CTR     │ Conversions      │
│  ─────────────────────────────────────────────────────────────────────────  │
│  text-slate-400 (header)      border-white/10                              │
│                                                                             │
│  Quiz "Let us help"          │ 30    │ 12     │ 40.0%   │ 3 (25%)         │
│  bg-white/5 hover:bg-white/10                   text-emerald-400          │
│                                                                             │
│  Hero "Get Free Check"       │ 120   │ 15     │ 12.5%   │ 8 (53%)         │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

**Interactions:**
- Sortable columns (click header)
- Hover row highlight

### 5.3 Time to Conversion (FR-3)

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ Time to Conversion                                                          │
├───────────────────────┬───────────────────────┬─────────────────────────────┤
│                       │                       │                             │
│  Avg. Time to         │  Avg. Time to         │  Returning Visitor          │
│  First Action         │  Paid Conversion      │  Conversion Rate            │
│                       │                       │                             │
│  2.3 min              │  1.4 days             │  4.2%                       │
│  text-2xl font-bold   │  text-2xl font-bold   │  text-2xl font-bold         │
│  text-cyan-400        │  text-cyan-400        │  text-emerald-400           │
│                       │                       │                             │
│  first visit →        │  first visit →        │  vs 0.8% new                │
│  CTA click            │  purchase             │  text-slate-500             │
│                       │                       │                             │
├───────────────────────┴───────────────────────┴─────────────────────────────┤
│                                                                             │
│  Distribution: Time to Purchase                                             │
│  ██████████████████████████ Same session (45%)    bg-cyan-500              │
│  ████████████ 1-24 hours (22%)                                             │
│  ██████ 1-3 days (15%)                                                     │
│  ███ 3-7 days (10%)                                                        │
│  ██ 7+ days (8%)                                                           │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 5.4 Returning vs. New (FR-4)

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ Visitor Segments                                                            │
├─────────────────────────────────┬───────────────────────────────────────────┤
│                                 │                                           │
│  NEW VISITORS                   │  RETURNING VISITORS                       │
│  text-slate-400 uppercase       │  text-slate-400 uppercase                 │
│                                 │                                           │
│  340 visitors                   │  45 visitors                              │
│  text-3xl font-bold             │  text-3xl font-bold                       │
│                                 │                                           │
│  2 conversions                  │  3 conversions                            │
│  0.6% rate                      │  6.7% rate                                │
│  text-slate-500                 │  text-emerald-400                         │
│                                 │                                           │
│                                 │  ████████████ 11x better                  │
│                                 │  text-emerald-400                         │
│                                 │                                           │
└─────────────────────────────────┴───────────────────────────────────────────┘
```

---

## 6. Query Specifications

### 6.1 CTA Performance Query

```typescript
// lib/queries/cta.ts

export async function getCtaPerformance(
  projectId: string,
  dateRange: { start: Date; end: Date }
) {
  const data = await sql`
    WITH cta_views AS (
      SELECT
        event_data->>'button_name' as cta_name,
        COUNT(*) as views,
        COUNT(DISTINCT device_id) as unique_views
      FROM events
      WHERE project_id = ${projectId}
        AND type = 'cta_view'
        AND timestamp >= ${dateRange.start}
        AND timestamp < ${dateRange.end}
      GROUP BY event_data->>'button_name'
    ),
    cta_clicks AS (
      SELECT
        COALESCE(event_data->>'button_name', event_data->>'location') as cta_name,
        COUNT(*) as clicks,
        COUNT(DISTINCT device_id) as unique_clicks
      FROM events
      WHERE project_id = ${projectId}
        AND type = 'cta_click'
        AND timestamp >= ${dateRange.start}
        AND timestamp < ${dateRange.end}
      GROUP BY COALESCE(event_data->>'button_name', event_data->>'location')
    ),
    conversions AS (
      SELECT DISTINCT device_id
      FROM events
      WHERE project_id = ${projectId}
        AND type IN ('checkout_complete', 'donate_complete', 'quick_check_complete')
        AND timestamp >= ${dateRange.start}
        AND timestamp < ${dateRange.end}
    )
    SELECT
      v.cta_name,
      COALESCE(v.views, 0) as views,
      COALESCE(c.clicks, 0) as clicks,
      CASE WHEN v.views > 0
        THEN ROUND((c.clicks::numeric / v.views) * 100, 1)
        ELSE 0
      END as ctr,
      COUNT(DISTINCT conv.device_id) as conversions
    FROM cta_views v
    LEFT JOIN cta_clicks c ON v.cta_name = c.cta_name
    LEFT JOIN conversions conv ON TRUE  -- Simplified, needs device correlation
    GROUP BY v.cta_name, v.views, c.clicks
    ORDER BY ctr DESC
  `;

  return data;
}
```

### 6.2 Time to Conversion Query

```typescript
// lib/queries/conversions.ts

export async function getTimeToConversion(
  projectId: string,
  dateRange: { start: Date; end: Date }
) {
  // Average time from first visit to first CTA click
  const timeToAction = await sql`
    WITH first_visits AS (
      SELECT device_id, MIN(timestamp) as first_visit
      FROM events
      WHERE project_id = ${projectId}
        AND timestamp >= ${dateRange.start}
        AND timestamp < ${dateRange.end}
      GROUP BY device_id
    ),
    first_actions AS (
      SELECT device_id, MIN(timestamp) as first_action
      FROM events
      WHERE project_id = ${projectId}
        AND type = 'cta_click'
        AND timestamp >= ${dateRange.start}
        AND timestamp < ${dateRange.end}
      GROUP BY device_id
    )
    SELECT AVG(EXTRACT(EPOCH FROM (fa.first_action - fv.first_visit))) as avg_seconds
    FROM first_visits fv
    JOIN first_actions fa ON fv.device_id = fa.device_id
  `;

  // Average time from first visit to purchase
  const timeToPurchase = await sql`
    WITH first_visits AS (
      SELECT device_id, MIN(timestamp) as first_visit
      FROM events
      WHERE project_id = ${projectId}
      GROUP BY device_id
    ),
    purchases AS (
      SELECT device_id, MIN(timestamp) as purchase_time
      FROM events
      WHERE project_id = ${projectId}
        AND type IN ('checkout_complete', 'donate_complete')
        AND timestamp >= ${dateRange.start}
        AND timestamp < ${dateRange.end}
      GROUP BY device_id
    )
    SELECT AVG(EXTRACT(EPOCH FROM (p.purchase_time - fv.first_visit))) as avg_seconds
    FROM first_visits fv
    JOIN purchases p ON fv.device_id = p.device_id
  `;

  // Distribution buckets
  const distribution = await sql`
    WITH first_visits AS (
      SELECT device_id, MIN(timestamp) as first_visit
      FROM events
      WHERE project_id = ${projectId}
      GROUP BY device_id
    ),
    purchases AS (
      SELECT device_id, MIN(timestamp) as purchase_time
      FROM events
      WHERE project_id = ${projectId}
        AND type IN ('checkout_complete', 'donate_complete')
        AND timestamp >= ${dateRange.start}
        AND timestamp < ${dateRange.end}
      GROUP BY device_id
    ),
    deltas AS (
      SELECT EXTRACT(EPOCH FROM (p.purchase_time - fv.first_visit)) as seconds
      FROM first_visits fv
      JOIN purchases p ON fv.device_id = p.device_id
    )
    SELECT
      CASE
        WHEN seconds < 1800 THEN 'same_session'
        WHEN seconds < 86400 THEN '1_24_hours'
        WHEN seconds < 259200 THEN '1_3_days'
        WHEN seconds < 604800 THEN '3_7_days'
        ELSE '7_plus_days'
      END as bucket,
      COUNT(*) as count
    FROM deltas
    GROUP BY bucket
  `;

  return {
    avgTimeToAction: timeToAction[0]?.avg_seconds || 0,
    avgTimeToPurchase: timeToPurchase[0]?.avg_seconds || 0,
    distribution,
  };
}
```

### 6.3 Returning vs. New Query

```typescript
// lib/queries/conversions.ts

export async function getVisitorSegments(
  projectId: string,
  dateRange: { start: Date; end: Date }
) {
  const segments = await sql`
    WITH visitor_status AS (
      SELECT
        device_id,
        BOOL_OR(is_first_visit) as is_new
      FROM events
      WHERE project_id = ${projectId}
        AND timestamp >= ${dateRange.start}
        AND timestamp < ${dateRange.end}
      GROUP BY device_id
    ),
    conversions AS (
      SELECT DISTINCT device_id
      FROM events
      WHERE project_id = ${projectId}
        AND type IN ('checkout_complete', 'donate_complete', 'quick_check_complete')
        AND timestamp >= ${dateRange.start}
        AND timestamp < ${dateRange.end}
    )
    SELECT
      CASE WHEN vs.is_new THEN 'new' ELSE 'returning' END as segment,
      COUNT(DISTINCT vs.device_id) as visitors,
      COUNT(DISTINCT c.device_id) as conversions
    FROM visitor_status vs
    LEFT JOIN conversions c ON vs.device_id = c.device_id
    GROUP BY vs.is_new
  `;

  return segments;
}
```

---

## 7. Implementation Order

### 7.1 Phase 1a: Foundation (Week 1)

| Task | Effort | Dependencies |
|------|--------|--------------|
| Schema update (`is_first_visit`) | 2h | None |
| Update `/api/events` for first visit detection | 2h | Schema |
| Backfill existing data | 1h | Schema |
| Create `lib/config/funnels.ts` | 1h | None |
| Create Tab routing in page.tsx | 2h | None |
| Extract OverviewTab from existing code | 2h | Tab routing |

### 7.2 Phase 1b: Funnel (Week 1-2)

| Task | Effort | Dependencies |
|------|--------|--------------|
| Create `lib/queries/funnel.ts` | 3h | Funnel config |
| Create FunnelTab component | 4h | Queries |
| Create FunnelChart component | 4h | None |
| Test with ChainSights data | 2h | FR-0 complete |

### 7.3 Phase 1c: CTA Performance (Week 2)

| Task | Effort | Dependencies |
|------|--------|--------------|
| Create `lib/queries/cta.ts` | 3h | None |
| Create CtaTab component | 3h | Queries |
| Sortable table implementation | 2h | None |

### 7.4 Phase 1d: Conversions (Week 2-3)

| Task | Effort | Dependencies |
|------|--------|--------------|
| Create `lib/queries/conversions.ts` | 4h | is_first_visit |
| Create ConversionsTab component | 4h | Queries |
| Time stats cards | 2h | None |
| Distribution histogram | 2h | None |
| Visitor segment cards | 2h | None |

### 7.5 Total Estimated Effort

| Phase | Hours |
|-------|-------|
| Foundation | 10h |
| Funnel | 13h |
| CTA | 8h |
| Conversions | 14h |
| **Total** | **~45h** |

---

## 8. Testing Strategy

### 8.1 Data Validation

```sql
-- Verify funnel data makes sense
SELECT
  type,
  COUNT(*) as count,
  COUNT(DISTINCT device_id) as unique_devices
FROM events
WHERE project_id = 'chainsights'
  AND timestamp >= NOW() - INTERVAL '7 days'
GROUP BY type
ORDER BY count DESC;
```

### 8.2 Component Testing

- Verify tabs switch correctly
- Verify date range applies to all tabs
- Verify empty states display properly
- Verify mobile responsiveness

### 8.3 Query Performance

- Add `EXPLAIN ANALYZE` to queries during development
- Target < 500ms for all dashboard queries
- Add indexes if needed

---

## 9. Rollout Plan

1. **Deploy schema update** (can be done independently)
2. **Deploy tab infrastructure** with only Overview tab active
3. **Enable Funnel tab** once FR-0 is confirmed working
4. **Enable CTA tab**
5. **Enable Conversions tab**

Each tab can be feature-flagged via config if needed.

---

## 10. Open Questions

| Question | Owner | Status |
|----------|-------|--------|
| Should funnel show unique visitors or total events? | Mary | **Decision: Unique (device_id)** |
| CTA naming consistency across projects? | Sempre | Open |
| Time to conversion: from first visit ever, or first visit in range? | Winston | **Decision: First visit ever** |

---

## 11. Additional Feature: Intraday Chart for Single Day

### 11.1 Requirement

When filter is set to **"Today"** or any **single day**, the "Views over Time" chart should display **hourly data** instead of a single daily bar.

### 11.2 Behavior Matrix

| Range | Granularity | X-Axis Labels | Bars |
|-------|-------------|---------------|------|
| 1 day (Today) | Hourly | 00, 02, 04, ..., 22 | 24 |
| 2-14 days | Daily | Mo, Di, Mi... or dates | N days |
| 15-90 days | Daily | Dates | N days |
| 90+ days | Weekly (optional) | KW01, KW02... | N weeks |

### 11.3 UI Specification

**Single Day (Hourly):**
```
┌─────────────────────────────────────────────────────────────────────────────┐
│ Views over Time                                              Today ▼       │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  45 ┤                     ██                                                │
│     │                     ██                                                │
│  30 ┤              ██     ██ ██                                             │
│     │           ██ ██ ██  ██ ██ ██                                          │
│  15 ┤     ██    ██ ██ ██  ██ ██ ██ ██                                       │
│     │  ██ ██ ██ ██ ██ ██  ██ ██ ██ ██ ██                                    │
│   0 ┼──────────────────────────────────────────────────────────────────     │
│     00   04   08   12   16   20   24                                        │
│     text-slate-500 text-xs                                                  │
│                                                                             │
│  Peak: 14:00 (45 views)                    text-slate-400                   │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 11.4 Query Changes

```typescript
// lib/queries/views-over-time.ts

export async function getViewsOverTime(
  projectId: string,
  dateRange: { start: Date; end: Date }
) {
  const daysDiff = Math.ceil(
    (dateRange.end.getTime() - dateRange.start.getTime()) / (1000 * 60 * 60 * 24)
  );

  // Single day → hourly granularity
  if (daysDiff <= 1) {
    return await sql`
      SELECT
        DATE_TRUNC('hour', timestamp AT TIME ZONE 'Europe/Vienna') as bucket,
        EXTRACT(HOUR FROM timestamp AT TIME ZONE 'Europe/Vienna') as hour,
        COUNT(*) as views,
        COUNT(DISTINCT device_id) as visitors
      FROM events
      WHERE project_id = ${projectId}
        AND timestamp >= ${dateRange.start}
        AND timestamp < ${dateRange.end}
        AND (type IS NULL OR type = 'pageView')
      GROUP BY bucket, hour
      ORDER BY bucket
    `;
  }

  // Multiple days → daily granularity (existing behavior)
  return await sql`
    SELECT
      DATE(timestamp AT TIME ZONE 'Europe/Vienna') as bucket,
      COUNT(*) as views,
      COUNT(DISTINCT device_id) as visitors
    FROM events
    WHERE project_id = ${projectId}
      AND timestamp >= ${dateRange.start}
      AND timestamp < ${dateRange.end}
      AND (type IS NULL OR type = 'pageView')
    GROUP BY bucket
    ORDER BY bucket
  `;
}
```

### 11.5 Component Update

```typescript
// In ViewsOverTimeChart component

interface ChartData {
  bucket: string;
  views: number;
  visitors: number;
  hour?: number;  // Only present for hourly data
}

function formatXLabel(data: ChartData, isHourly: boolean): string {
  if (isHourly && data.hour !== undefined) {
    return `${data.hour.toString().padStart(2, '0')}:00`;
  }
  // Daily: format as weekday or date
  return new Date(data.bucket).toLocaleDateString('de-AT', {
    weekday: 'short',
    day: 'numeric',
  });
}

// Detect hourly mode
const isHourly = data.length > 0 && data[0].hour !== undefined;
```

### 11.6 Implementation Effort

| Task | Effort |
|------|--------|
| Update query logic with granularity detection | 1h |
| Update chart component for hourly labels | 1h |
| Add "Peak hour" indicator (optional) | 0.5h |
| **Total** | **2.5h** |

---

*Tech Spec created by the Analytics Party Team*
*Winston (Architect), Mary (Analyst), Sally (UX)*
