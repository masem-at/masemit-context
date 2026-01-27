# Story 12.5: Add Anomaly Alert Card to Admin Dashboard

Status: done

## Story

As an **admin**,
I want **a dashboard card showing recent score anomalies**,
So that I can **see significant changes at a glance** without checking email/Slack.

## Acceptance Criteria

### AC1: Anomaly Alert Card Display
- [x] Card appears in admin dashboard Overview section
- [x] Card title: "Score Anomalies (24h)" with warning icon
- [x] Shows count of anomalies in card header badge

### AC2: Anomaly List Content
- [x] Each anomaly shows: DAO name, old score → new score, delta
- [x] Color coding: red background/text for drops, green for increases
- [x] Timestamp of detection shown
- [x] Maximum 10 most recent anomalies displayed

### AC3: Empty State
- [x] When no anomalies in 24h: "No significant changes detected"
- [x] Card has neutral/green status indicator
- [x] Empty state is visually distinct but not alarming

### AC4: Card Styling
- [x] Matches existing admin dashboard card styling (shadcn/ui)
- [x] Warning border when anomalies present (orange/yellow)
- [x] Neutral border when no anomalies

### AC5: Data Freshness
- [x] Card data fetched server-side (no client polling)
- [x] Refreshes on page load
- [x] Shows "Last checked: [timestamp]" footer

## Tasks / Subtasks

- [x] Task 1: Create Data Fetching Function (AC: #2, #5)
  - [x] 1.1 Implement `getRecentAnomalies(hours: number)` in job-logs.ts (Story 12-4)
  - [x] 1.2 Query job_logs for last 24h with database-level filtering
  - [x] 1.3 Return array of anomaly objects sorted by detection time (newest first)
  - [x] 1.4 Add type definitions for anomaly data (AnomalyWithContext)

- [x] Task 2: Create Anomaly Alert Card Component (AC: #1, #2, #3, #4)
  - [x] 2.1 Create `src/components/admin/anomaly-alerts.tsx`
  - [x] 2.2 Accept anomalies array as prop
  - [x] 2.3 Render card with header, badge count, and list
  - [x] 2.4 Implement empty state UI
  - [x] 2.5 Add color coding for up/down changes

- [x] Task 3: Integrate into Admin Dashboard (AC: #1, #5)
  - [x] 3.1 Import and use component in `src/app/admin/page.tsx`
  - [x] 3.2 Fetch anomalies in server component
  - [x] 3.3 Position card in Overview section (3-column grid layout)
  - [x] 3.4 Pass data to component

- [x] Task 4: Styling & Polish (AC: #4)
  - [x] 4.1 Add warning border style when anomalies present (orange-500/50)
  - [x] 4.2 Match existing card styling patterns (shadcn/ui Card)
  - [x] 4.3 Ensure responsive layout on mobile (grid-cols-1 lg:grid-cols-3)
  - [x] 4.4 Add hover states for anomaly rows (hover:bg-*-500/15)

- [x] Task 5: Testing (AC: #1-5)
  - [x] 5.1 Unit test component with mock anomaly data (17 tests)
  - [x] 5.2 Unit test empty state rendering
  - [x] 5.3 Visual test in Storybook (if available) - N/A
  - [x] 5.4 Integration test with real data fetch - covered by server component

## Dev Notes

### Files to Create/Modify

```
src/components/admin/anomaly-alerts.tsx   # NEW - card component
src/lib/gvs/anomaly-detection.ts          # Add getRecentAnomalies()
src/app/admin/page.tsx                    # Integrate card
```

### Component Structure

```tsx
// src/components/admin/anomaly-alerts.tsx

import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/card'
import { Badge } from '@/components/ui/badge'
import { AlertTriangle, TrendingDown, TrendingUp, CheckCircle } from 'lucide-react'

interface GVSAnomaly {
  daoId: string
  daoName: string
  previousScore: number
  currentScore: number
  delta: number
  direction: 'up' | 'down'
  detectedAt: Date
}

interface AnomalyAlertsProps {
  anomalies: GVSAnomaly[]
  lastChecked: Date
}

export function AnomalyAlerts({ anomalies, lastChecked }: AnomalyAlertsProps) {
  const hasAnomalies = anomalies.length > 0

  return (
    <Card className={`border-border bg-card ${hasAnomalies ? 'border-orange-500/50' : ''}`}>
      <CardHeader className="pb-3">
        <div className="flex items-center justify-between">
          <CardTitle className="text-white flex items-center gap-2">
            {hasAnomalies ? (
              <AlertTriangle className="w-5 h-5 text-orange-400" />
            ) : (
              <CheckCircle className="w-5 h-5 text-green-400" />
            )}
            Score Anomalies (24h)
          </CardTitle>
          {hasAnomalies && (
            <Badge className="bg-orange-500/20 text-orange-400 border-orange-500/30">
              {anomalies.length}
            </Badge>
          )}
        </div>
      </CardHeader>
      <CardContent>
        {hasAnomalies ? (
          <div className="space-y-2">
            {anomalies.slice(0, 10).map((anomaly, index) => (
              <AnomalyRow key={index} anomaly={anomaly} />
            ))}
          </div>
        ) : (
          <div className="text-center py-4 text-gray-400">
            No significant changes detected
          </div>
        )}
        <div className="text-xs text-gray-500 mt-3 pt-3 border-t border-border">
          Last checked: {lastChecked.toLocaleString()}
        </div>
      </CardContent>
    </Card>
  )
}

function AnomalyRow({ anomaly }: { anomaly: GVSAnomaly }) {
  const isDown = anomaly.direction === 'down'

  return (
    <div className={`flex items-center justify-between p-3 rounded-lg border ${
      isDown
        ? 'border-red-500/30 bg-red-500/10'
        : 'border-green-500/30 bg-green-500/10'
    }`}>
      <div className="flex items-center gap-3">
        {isDown ? (
          <TrendingDown className="w-4 h-4 text-red-400" />
        ) : (
          <TrendingUp className="w-4 h-4 text-green-400" />
        )}
        <div>
          <div className="text-sm font-medium text-white">{anomaly.daoName}</div>
          <div className="text-xs text-gray-400">
            {new Date(anomaly.detectedAt).toLocaleTimeString()}
          </div>
        </div>
      </div>
      <div className="text-right">
        <div className={`text-sm font-medium ${isDown ? 'text-red-400' : 'text-green-400'}`}>
          {anomaly.previousScore.toFixed(1)} → {anomaly.currentScore.toFixed(1)}
        </div>
        <div className={`text-xs ${isDown ? 'text-red-400' : 'text-green-400'}`}>
          {anomaly.delta > 0 ? '+' : ''}{anomaly.delta.toFixed(1)}
        </div>
      </div>
    </div>
  )
}
```

### Admin Page Integration

```tsx
// In src/app/admin/page.tsx

import { AnomalyAlerts } from '@/components/admin/anomaly-alerts'
import { getRecentAnomalies } from '@/lib/gvs/anomaly-detection'

// In the server component:
const anomalies = await getRecentAnomalies(24) // Last 24 hours

// In the JSX, after status overview:
<AnomalyAlerts anomalies={anomalies} lastChecked={new Date()} />
```

### Card Placement

Position the anomaly card prominently in the Overview section:

```
┌─────────────────────────────────────────────────────────┐
│ Admin Dashboard                                          │
├─────────────────────────────────────────────────────────┤
│ [Stats Cards: Total Orders | Ready | Delivered | etc.]  │
├─────────────────────────────────────────────────────────┤
│ ┌─────────────────────┐  ┌─────────────────────┐       │
│ │ ⚠️ Score Anomalies  │  │ Weekly GVS Jobs     │       │
│ │ (24h)          [3]  │  │                     │       │
│ │                     │  │                     │       │
│ │ Lido: 7.5→5.8 -1.7  │  │ 10/10 DAOs ✓       │       │
│ │ Balancer: 5.0→4.2   │  │ ...                 │       │
│ └─────────────────────┘  └─────────────────────┘       │
└─────────────────────────────────────────────────────────┘
```

### Dependency on Story 12-4

This story requires Story 12-4 (anomaly detection) to be completed first, as it provides the `getRecentAnomalies()` function and anomaly data storage.

### Testing Strategy

1. **Unit Tests:**
   - Component renders with anomalies
   - Component renders empty state
   - Color coding correct for up/down
   - Badge count matches anomaly count

2. **Visual Tests:**
   - Screenshot test for anomaly state
   - Screenshot test for empty state
   - Mobile responsiveness

### References

- [Source: src/app/admin/page.tsx] - Current admin dashboard structure
- [Source: src/components/admin/recalculate-controls.tsx] - Similar card pattern
- [Design: shadcn/ui Card component] - Styling reference

## Dev Agent Record

### Context Reference

Story created from Epic 12: Daily GVS Monitoring & Historical Tracking.
Depends on: Story 12-4 (Anomaly Detection with Alerts)

### Agent Model Used

Claude Opus 4.5 (claude-opus-4-5-20251101)

### Debug Log References

None

### Completion Notes List

- All 5 Acceptance Criteria implemented and verified
- Card displays in 3-column grid layout with job logs and opt-out cards
- Color coding uses red for drops (border-red-500/30, bg-red-500/10) and green for increases
- Warning border (border-orange-500/50) applied when anomalies present
- Maximum 10 anomalies displayed with "+N more" indicator
- Hover states added for better interactivity
- 17 unit tests all passing
- Reuses getRecentAnomalies() from Story 12-4 (job-logs.ts)
- Adversarial code review performed and all issues fixed

### File List

- `src/components/admin/anomaly-alerts.tsx` (NEW) - AnomalyAlerts component with AnomalyRow subcomponent
- `src/app/admin/page.tsx` (MODIFIED) - Integrated anomaly card in 3-column grid layout
- `tests/unit/components/anomaly-alerts.test.tsx` (NEW) - 17 unit tests for component
