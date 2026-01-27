# Feature Spec: Pipeline Status UX Improvements

**Date:** 2025-12-17
**Author:** Sally (UX Designer)
**Status:** Draft
**Priority:** High

---

## 1. Problem Statement

The current Pipeline Status section uses identical visual treatment for "in progress" and "waiting" states, leaving users confused about whether the pipeline is actively running or stuck.

**User Story:**
> *"I submitted a report 2 minutes ago. I see gray badges with refresh icons everywhere. Is it working? Is it frozen? Should I click something? I keep refreshing the page hoping something changes."*

---

## 2. Current Behavior

| State | Current Visual | Problem |
|-------|----------------|---------|
| Completed | ✅ Green check | Works well |
| In Progress | 🔄 Gray refresh icon | Same as "waiting" |
| Waiting | 🔄 Gray refresh icon | Same as "in progress" |
| Failed | 🔄 Gray refresh icon | No indication of failure |
| Needs Action | ⚠️ Aqua alert (review only) | Only for review step |

**Screenshot reference:** `docs/_masemIT/Screenshot 2025-12-17 111233.png`

---

## 3. Proposed Solution

Create a clear visual language with **4 distinct states** and add runtime information.

### Enhanced Pipeline Status Card

```
┌─────────────────────────────────────────────────────────────┐
│ Pipeline Status                                             │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ✅ ──────── 🔄 ──────── ○ ──────── ○ ──────── ○           │
│  Fetched    Analyzing   PDF       Review     Send           │
│              ↑                                              │
│         Currently running                                   │
│                                                             │
│  ⏱️ Started 1m 23s ago                                      │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Visual State Language

| State | Icon | Color | Animation | Label Style |
|-------|------|-------|-----------|-------------|
| **Completed** | `CheckCircle` | Green (`#22c55e`) | None | Normal |
| **In Progress** | `Loader2` | Blue/Aqua (`#00E5C0`) | `animate-spin` | **Bold** + "Currently running" |
| **Waiting** | `Circle` (outline) | Gray (`#6b7280`) | None | Muted |
| **Failed** | `XCircle` | Red (`#ef4444`) | None | "Failed" label |
| **Needs Action** | `AlertCircle` | Aqua (`#00E5C0`) | `animate-pulse` | "Awaiting review" |

---

## 4. Requirements

| # | Requirement | Priority |
|---|-------------|----------|
| 1 | Differentiate "in progress" from "waiting" with animated spinner | Must |
| 2 | Show "waiting" steps as outline circles (not filled) | Must |
| 3 | Show elapsed time when pipeline is running | Must |
| 4 | Show failed step with red X icon | Must |
| 5 | Highlight current step with label underneath | Should |
| 6 | Auto-refresh status every 5 seconds when running | Should |
| 7 | Show estimated time remaining (optional) | Could |

---

## 5. State Detection Logic

```typescript
type StepState = 'completed' | 'running' | 'waiting' | 'failed' | 'needs_action'

function getStepState(report: Report, step: string): StepState {
  const runningStatuses = ['fetching', 'analyzing', 'generating']
  const isRunning = runningStatuses.includes(report.status)

  switch (step) {
    case 'fetch':
      if (report.rawData) return 'completed'
      if (report.status === 'fetching') return 'running'
      if (report.status === 'failed' && !report.rawData) return 'failed'
      return 'waiting'

    case 'analyze':
      if (report.aiAnalysis) return 'completed'
      if (report.status === 'analyzing') return 'running'
      if (report.status === 'failed' && report.rawData && !report.aiAnalysis) return 'failed'
      return 'waiting'

    case 'generate':
      if (report.finalPdfUrl) return 'completed'
      if (report.status === 'generating') return 'running'
      if (report.status === 'failed' && report.aiAnalysis && !report.finalPdfUrl) return 'failed'
      return 'waiting'

    case 'review':
      if (report.approvedAt) return 'completed'
      if (report.status === 'review_pending' || report.status === 'declined') return 'needs_action'
      return 'waiting'

    case 'send':
      if (report.deliveredAt) return 'completed'
      return 'waiting'

    default:
      return 'waiting'
  }
}
```

---

## 6. Elapsed Time Display

Show elapsed time when any step is "running":

```typescript
function getElapsedTime(report: Report): string | null {
  const runningStatuses = ['fetching', 'analyzing', 'generating']
  if (!runningStatuses.includes(report.status)) return null

  const startTime = report.updatedAt // Last status change
  const elapsed = Date.now() - new Date(startTime).getTime()

  const seconds = Math.floor(elapsed / 1000)
  const minutes = Math.floor(seconds / 60)

  if (minutes > 0) {
    return `${minutes}m ${seconds % 60}s`
  }
  return `${seconds}s`
}
```

---

## 7. UI Mockups

### State: Running (Analyzing)

```
┌─────────────────────────────────────────────────────────────┐
│ Pipeline Status                                             │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ✅ ─────── 🔄 ─────── ○ ─────── ○ ─────── ○               │
│  Fetched   Analyzing   PDF      Review    Send              │
│            (spinning)                                       │
│                                                             │
│  🔄 Analyzing governance data...                            │
│  ⏱️ Running for 45s                                         │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### State: Awaiting Review

```
┌─────────────────────────────────────────────────────────────┐
│ Pipeline Status                                             │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ✅ ─────── ✅ ─────── ✅ ─────── ⚠️ ─────── ○              │
│  Fetched   Analyzed    PDF       Review    Send             │
│                                 (pulsing)                   │
│                                                             │
│  ⚠️ Awaiting your review                                    │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### State: Failed

```
┌─────────────────────────────────────────────────────────────┐
│ Pipeline Status                                             │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ❌ ─────── ○ ─────── ○ ─────── ○ ─────── ○                │
│  Fetch     Analyze    PDF      Review    Send               │
│  FAILED                                                     │
│                                                             │
│  ❌ Pipeline failed at Data Fetch                           │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 8. Auto-Refresh Behavior

When pipeline is running, auto-refresh status:

```typescript
useEffect(() => {
  const runningStatuses = ['pending', 'fetching', 'analyzing', 'generating']

  if (report && runningStatuses.includes(report.status)) {
    const interval = setInterval(async () => {
      const response = await fetch(`/api/admin/reports/${report.id}`)
      if (response.ok) {
        const updated = await response.json()
        setReport(updated)
      }
    }, 5000) // Refresh every 5 seconds

    return () => clearInterval(interval)
  }
}, [report?.status, report?.id])
```

---

## 9. Technical Notes

**File to modify:** `src/app/admin/reports/[id]/page.tsx`

**New icons needed:**
- `Circle` (outline) — for waiting state
- `XCircle` — for failed state (already imported but unused here)

**Component structure:**
```tsx
function PipelineStep({ state, label }: { state: StepState, label: string }) {
  const icons = {
    completed: <CheckCircle className="w-5 h-5 text-green-400" />,
    running: <Loader2 className="w-5 h-5 text-aqua animate-spin" />,
    waiting: <Circle className="w-5 h-5 text-gray-500" />,
    failed: <XCircle className="w-5 h-5 text-red-400" />,
    needs_action: <AlertCircle className="w-5 h-5 text-aqua animate-pulse" />,
  }

  return (
    <div className="flex flex-col items-center gap-1">
      {icons[state]}
      <span className={`text-xs ${state === 'running' ? 'font-bold text-aqua' : 'text-gray-400'}`}>
        {label}
      </span>
    </div>
  )
}
```

---

## 10. Acceptance Criteria

- [ ] "In progress" steps show animated spinner icon
- [ ] "Waiting" steps show outline circle (distinct from in-progress)
- [ ] "Failed" steps show red X icon
- [ ] Current running step is highlighted with bold label
- [ ] Elapsed time is displayed when pipeline is running
- [ ] Status auto-refreshes every 5 seconds during running states
- [ ] "Awaiting review" shows pulsing alert icon

---

*Document created by Sally (UX Designer) — 2025-12-17*
