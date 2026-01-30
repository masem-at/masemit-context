# Feature Spec: Error Details Display in Admin UI

**Date:** 2025-12-17
**Author:** Mario (via Mary, Business Analyst)
**UX Review:** Sally (UX Designer) — 2025-12-17
**Status:** Draft
**Priority:** Medium

---

## 1. Problem Statement

When a report generation fails, the admin UI shows only a "failed" status badge without any information about what went wrong. The error message is stored in the database (`reports.errorMessage`) but not displayed, making troubleshooting difficult.

**User Story:**
> *"I see a red 'failed' badge but have no idea what happened. Did the API timeout? Is the space ID wrong? I feel helpless and have to dig through logs to find out."*

---

## 2. Current Behavior

- Status badge shows "failed" (red)
- No error details visible
- Admin has no way to know:
  - Which pipeline step failed
  - What the error was
  - Whether it's recoverable
  - What action to take next

---

## 3. Proposed Solution

Add an **Error Details Card** that provides context, empathy, and clear next steps.

### UI Mockup

```
┌─────────────────────────────────────────────────────────────┐
│ ❌ Pipeline Failed at: Data Fetch                           │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ● ──────── ○ ──────── ○ ──────── ○ ──────── ○             │
│  Fetch     Analyze    PDF       Review     Send             │
│  ↑ FAILED                                                   │
│                                                             │
│ Error Message:                                              │
│ ┌─────────────────────────────────────────────────────────┐ │
│ │ Snapshot space "gitcoin.eth" not found or returned no   │ │
│ │ proposals within the specified timeframe.               │ │
│ └─────────────────────────────────────────────────────────┘ │
│                                                             │
│ 💡 Tip: This usually means the space ID is incorrect or    │
│    the DAO has no recent governance activity to analyze.    │
│                                                             │
│ [🔄 Retry from Failed Step]  [🗑️ Reset Pipeline]            │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Visual States

| Failed Step | Visual Indicator |
|-------------|------------------|
| Fetch | Red dot on "Fetch", gray circles for rest |
| Analyze | Green check on Fetch, red dot on Analyze, gray rest |
| PDF Generation | Green checks on Fetch/Analyze, red dot on PDF |

---

## 4. Requirements

| # | Requirement | Priority |
|---|-------------|----------|
| 1 | Show error card when `status === 'failed'` | Must |
| 2 | Display `errorMessage` from database | Must |
| 3 | Show which pipeline step failed (visual indicator) | Must |
| 4 | Style as error state (red accent, clear hierarchy) | Must |
| 5 | Provide contextual help tip based on error type | Should |
| 6 | Allow retry from failed step | Should |
| 7 | Allow full pipeline reset (clear data, start over) | Could |

---

## 5. Contextual Help Tips

Map common errors to helpful suggestions:

| Error Pattern | Tip |
|---------------|-----|
| "not found" / "no proposals" | "The space ID may be incorrect or the DAO has no recent activity." |
| "timeout" / "ETIMEDOUT" | "The Snapshot API is slow. Try again in a few minutes." |
| "rate limit" | "Too many requests. Wait 60 seconds before retrying." |
| "API key" / "unauthorized" | "Check your API credentials in environment variables." |
| Default | "Check the logs for more details or contact support." |

---

## 6. Technical Notes

**File to modify:** `src/app/admin/reports/[id]/page.tsx`

**Data available:**
- `report.errorMessage` — The error text
- `report.status` — To check if failed
- `report.rawData` — To determine if Fetch completed
- `report.aiAnalysis` — To determine if Analyze completed
- `report.finalPdfUrl` — To determine if PDF generation completed

**Determine failed step:**
```typescript
function getFailedStep(report: Report): string {
  if (!report.rawData) return 'fetch'
  if (!report.aiAnalysis) return 'analyze'
  if (!report.finalPdfUrl) return 'generate'
  return 'unknown'
}
```

**Implementation:**
```tsx
{report.status === 'failed' && (
  <Card className="border-red-500/30 bg-red-500/5 mb-6">
    <CardHeader>
      <CardTitle className="text-red-400 flex items-center gap-2">
        <XCircle className="w-5 h-5" />
        Pipeline Failed at: {getFailedStepLabel(report)}
      </CardTitle>
    </CardHeader>
    <CardContent className="space-y-4">
      {/* Mini pipeline showing failed step */}
      <div className="flex items-center gap-2">
        <StepIndicator done={!!report.rawData} failed={!report.rawData && report.status === 'failed'} label="Fetch" />
        <span className="text-gray-600">→</span>
        <StepIndicator done={!!report.aiAnalysis} failed={!!report.rawData && !report.aiAnalysis && report.status === 'failed'} label="Analyze" />
        {/* ... etc */}
      </div>

      {/* Error message */}
      {report.errorMessage && (
        <pre className="bg-navy-dark p-4 rounded-lg text-red-300 text-sm whitespace-pre-wrap border border-red-500/20">
          {report.errorMessage}
        </pre>
      )}

      {/* Contextual tip */}
      <div className="flex items-start gap-2 text-sm text-gray-400">
        <Lightbulb className="w-4 h-4 text-yellow-500 mt-0.5" />
        <span>{getErrorTip(report.errorMessage)}</span>
      </div>

      {/* Actions */}
      <div className="flex gap-3">
        <Button variant="outline" onClick={() => handleRetryFromFailed()}>
          <RefreshCw className="w-4 h-4 mr-2" />
          Retry from Failed Step
        </Button>
      </div>
    </CardContent>
  </Card>
)}
```

---

## 7. Acceptance Criteria

- [ ] Error card displays when report status is "failed"
- [ ] Error message from database is shown
- [ ] Failed step is visually indicated in mini-pipeline
- [ ] Card has appropriate error styling (red accents)
- [ ] Contextual tip is displayed based on error pattern
- [ ] Retry button allows re-running from failed step
- [ ] Card is positioned prominently (after Pipeline Status, before Actions)

---

*Document created by Mary (Business Analyst) — 2025-12-17*
*UX enhancements by Sally (UX Designer) — 2025-12-17*
