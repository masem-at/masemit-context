# Story: Loading Page UX Improvements

**Status:** Ready for Development
**Priority:** Medium (Polish)
**Estimated Effort:** 1-2 hours
**UX Spec:** `docs/ux/loading-experience-spec.md`

---

## Overview

Sally reviewed the loading page implementation and found it's mostly well-implemented. A few improvements are needed to align with the UX specification and the new tier-based scenario system.

---

## Tasks

### Task 1: Fix Stage Labels to Match Backend

**File:** `app/generate/loading/page.tsx`

**Problem:** The current `TIER_MESSAGES` have 4 generic steps that don't match actual backend stages.

**Current:**
```typescript
largecap: [
  'Initializing enterprise data structures...',
  'Generating financial services portfolio...',
  'Building compliance frameworks...',
  'Calculating quarterly projections...'
]
```

**Should be (5 stages):**
```typescript
const STAGE_LABELS = {
  largecap: [
    'Setting up company structure...',           // stage1_generating (10%)
    'Generating business trends...',              // stage2_generating (30%)
    'Creating monthly transactions...',           // stage3 (50-90%)
    'Storing events in database...',              // database_insert (90%)
    'Running consistency validation...'           // validation (95%)
  ],
  midcap: [
    'Setting up manufacturing operations...',
    'Generating production trends...',
    'Creating monthly transactions...',
    'Storing events in database...',
    'Running consistency validation...'
  ],
  startup: [
    'Creating startup profile...',
    'Modeling growth trajectory...',
    'Generating monthly transactions...',
    'Storing events in database...',
    'Running consistency validation...'
  ]
}
```

**Fix stage mapping logic:**
```typescript
// Update the stage detection in the polling useEffect:
if (status.currentStage) {
  if (status.currentStage.includes('stage1')) setCurrentStep(0)
  else if (status.currentStage.includes('stage2')) setCurrentStep(1)
  else if (status.currentStage.includes('stage3')) setCurrentStep(2)
  else if (status.currentStage.includes('database')) setCurrentStep(3)
  else if (status.currentStage.includes('validation')) setCurrentStep(4)
}
```

---

### Task 2: Improve Title Display

**File:** `app/generate/loading/page.tsx`

**Problem:** Title shows raw tier ID like "Largecap Scenario" instead of friendly name.

**Current (line ~265):**
```tsx
Generating Your {scenarioType.charAt(0).toUpperCase() + scenarioType.slice(1)} Scenario
```

**Fix:**
```typescript
const TIER_DISPLAY_NAMES: Record<string, string> = {
  largecap: 'Finance / Large Cap',
  midcap: 'Industrials / Mid Cap',
  startup: 'Consumer Staples / Startup',
  // Legacy
  bakery: 'Bakery',
  hotel: 'Hotel',
  techStartup: 'Tech Startup'
}

// Then in JSX:
Generating Your {TIER_DISPLAY_NAMES[scenarioType] || scenarioType} Scenario
```

---

### Task 3: Add "Back to Home" Button on Error Screen

**File:** `app/generate/loading/page.tsx`

**Problem:** Error screen only has "Try Again" button. Should have secondary option.

**Current (line ~228-234):**
```tsx
<button onClick={handleRetry} className="...">
  Try Again
</button>
```

**Fix:**
```tsx
<div className="flex gap-3 justify-center">
  <button
    onClick={handleRetry}
    className="px-6 py-3 bg-red-600 text-white rounded-lg hover:bg-red-700 transition-colors font-medium"
  >
    Try Again
  </button>
  <button
    onClick={() => router.push('/')}
    className="px-6 py-3 bg-white text-gray-700 border border-gray-300 rounded-lg hover:bg-gray-50 transition-colors font-medium"
  >
    Back to Home
  </button>
</div>
```

---

### Task 4: Add Time Estimate Display (Optional)

**File:** `app/generate/loading/page.tsx`

**Problem:** Only shows elapsed time, not estimated remaining time.

**Add constant:**
```typescript
const TIER_DURATION_ESTIMATES: Record<string, number> = {
  largecap: 300,   // ~5 minutes
  midcap: 240,     // ~4 minutes
  startup: 180,    // ~3 minutes
  bakery: 120,     // ~2 minutes
  hotel: 180,      // ~3 minutes
  techStartup: 300 // ~5 minutes
}
```

**Add to progress display:**
```tsx
<div className="flex justify-between text-sm text-gray-600">
  <span>{Math.round(progress)}% complete</span>
  <span className="font-mono">
    {elapsedSeconds}s elapsed
    {progress > 5 && progress < 95 && (
      <> / ~{Math.max(0, TIER_DURATION_ESTIMATES[scenarioType] - elapsedSeconds)}s remaining</>
    )}
  </span>
</div>
```

---

### Task 5: Add Cancel Confirmation Dialog (Optional)

**File:** `app/generate/loading/page.tsx`

**Problem:** Cancel navigates immediately without confirmation.

**Add state:**
```typescript
const [showCancelDialog, setShowCancelDialog] = useState(false)
```

**Update handleCancel:**
```typescript
const handleCancel = () => {
  setShowCancelDialog(true)
}

const confirmCancel = () => {
  router.push('/')
}
```

**Add dialog JSX (before closing div):**
```tsx
{showCancelDialog && (
  <div className="fixed inset-0 bg-black/50 flex items-center justify-center z-50">
    <div className="bg-white rounded-lg p-6 max-w-sm mx-4">
      <h3 className="text-lg font-semibold mb-2">Cancel Generation?</h3>
      <p className="text-gray-600 mb-4">
        Your scenario is still generating. You can check "My Scenarios" later to see if it completed.
      </p>
      <div className="flex gap-3 justify-end">
        <button
          onClick={() => setShowCancelDialog(false)}
          className="px-4 py-2 text-gray-600 hover:text-gray-800"
        >
          Keep Waiting
        </button>
        <button
          onClick={confirmCancel}
          className="px-4 py-2 bg-gray-600 text-white rounded-lg hover:bg-gray-700"
        >
          Yes, Cancel
        </button>
      </div>
    </div>
  </div>
)}
```

---

## Acceptance Criteria

- [ ] Stage labels match actual backend stages (5 steps)
- [ ] Title shows friendly tier names (not raw IDs)
- [ ] Error screen has both "Try Again" and "Back to Home" buttons
- [ ] (Optional) Time estimate shows remaining time
- [ ] (Optional) Cancel shows confirmation dialog

---

## Notes

- Tasks 1-3 are quick wins (~30 min total)
- Tasks 4-5 are nice-to-have polish
- No backend changes required - all frontend work
- Reference UX spec: `docs/ux/loading-experience-spec.md`
