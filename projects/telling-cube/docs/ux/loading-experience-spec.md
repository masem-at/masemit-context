# Loading Experience UX Specification

**Author:** Sally (UX Expert)
**Date:** 2025-12-09
**Last Updated:** 2025-12-10
**Status:** PARTIALLY IMPLEMENTED - Review Complete

---

## Implementation Review (2025-12-10)

### ✅ Already Implemented
1. ✅ Progress bar reflects actual backend progress (polling every 2s)
2. ✅ Stage checklist with visual indicators (checkmark, spinner, empty circle)
3. ✅ Error state with retry button
4. ✅ Cancel button (navigates home)
5. ✅ Educational tips carousel (rotates every 10s)
6. ✅ Value reinforcement card ("While you wait...")
7. ✅ Elapsed time counter
8. ✅ Success screen with auto-redirect
9. ✅ Accessibility: screen reader live region
10. ✅ New tier system support (largecap, midcap, startup)

### 🔶 Needs Improvement
1. 🔶 Stage labels don't match actual backend stages (see Task 1)
2. 🔶 No time estimate display (only elapsed shown)
3. 🔶 Cancel has no confirmation dialog
4. 🔶 Title shows "Largecap" instead of company name or friendly label

### ❌ Not Yet Implemented
1. ❌ "Back to Home" secondary button on error screen
2. ❌ Better stage mapping for monthly generation (12 months)

---

## Original Problem Statement

Users wait 2-8 minutes for scenario generation. Current implementation shows progress but I was never consulted on the design. Let's fix that.

**Original Issues (Now Mostly Resolved):**
1. ~~❌ Progress bar doesn't reflect actual backend progress~~ ✅ FIXED
2. ~~❌ No clear stage indicators for what's happening~~ ✅ FIXED
3. ~~❌ Error states not well-designed~~ 🔶 IMPROVED
4. ~~❌ No way to cancel or navigate away safely~~ 🔶 PARTIAL
5. ~~❌ Educational content feels disconnected from progress~~ ✅ FIXED

---

## Design Principles

### 1. Transparency
Users should always know:
- What's happening right now
- Why it's taking time
- Approximately how long is left

### 2. Reassurance
Long waits cause anxiety. We need to:
- Show continuous progress (never freeze)
- Explain the value being created
- Provide interesting content to pass time

### 3. Control
Users should feel in control:
- Clear cancel option
- Ability to navigate away (with warning)
- Resume if they leave and come back

### 4. Professionalism
This is a B2B SaaS tool:
- No silly animations or jokes
- Professional tone
- IBCS© standards language

---

## User Flows

### Flow 1: Successful Generation (Happy Path)

```
1. User clicks "Generate [Scenario]"
   ↓
2. Immediate feedback: "Starting generation..."
   ↓
3. Loading page appears within 200ms
   ↓
4. Progress bar starts at 0%
   ↓
5. Stage updates every 2-3 seconds:
   - "Setting up company structure..." (0-15%)
   - "Generating monthly business trends..." (15-30%)
   - "Creating January transactions..." (30-40%)
   - "Creating February transactions..." (40-50%)
   - ... (each month increments ~8%)
   - "Storing 153 events in database..." (90-95%)
   - "Running consistency validation..." (95-99%)
   - "Complete!" (100%)
   ↓
6. Brief success screen (0.8s)
   ↓
7. Auto-redirect to results
```

**Total time:** 2-8 minutes depending on scenario

---

### Flow 2: Generation Failure

```
1-4. Same as happy path
   ↓
5. Error detected at any stage
   ↓
6. Progress stops, error icon appears
   ↓
7. Error screen shows:
   - Clear error message
   - What went wrong
   - Two action buttons:
     * "Try Again" (primary)
     * "Back to Home" (secondary)
   ↓
8. User chooses action
```

**Error message examples:**
- "Generation took longer than expected. Please try again."
- "Network connection lost. Please check your connection and try again."
- "Something went wrong. Our team has been notified. Please try again in a few minutes."

---

### Flow 3: User Cancels

```
1-4. Same as happy path
   ↓
5. User clicks "Cancel"
   ↓
6. Confirmation dialog:
   "Are you sure? Your scenario is still generating.
    You can come back later to check progress."

   [Yes, Cancel]  [Keep Waiting]
   ↓
7a. If "Yes, Cancel" → Return to home
    (Backend continues, scenario still created)

7b. If "Keep Waiting" → Stay on page
```

**Note:** Generation continues in background (QStash handles this)

---

### Flow 4: User Closes Tab/Browser

```
1-4. Same as happy path
   ↓
5. User closes tab/browser
   ↓
6. Generation continues in background (QStash)
   ↓
7. User returns later
   ↓
8. Option A: Send email when complete (future enhancement)
   Option B: "My Scenarios" page shows in-progress status
```

**Current implementation:** Scenario completes but user doesn't know
**Proposed:** Add to roadmap

---

## Visual Design Specification

### Layout Structure

```
┌─────────────────────────────────────────┐
│         tellingCube.com                 │  ← Header (logo)
├─────────────────────────────────────────┤
│                                         │
│     Generating Your Tech Startup        │  ← Title
│              Scenario                   │
│                                         │
│  Creating 1 year of realistic business  │  ← Subtitle
│  data with mathematical consistency...  │
│                                         │
│  ═══════════════════ 47%                │  ← Progress bar
│  Estimated time remaining: 4 minutes    │  ← Time estimate
│                                         │
│  ┌─────────────────────────────────┐   │
│  │ ✓ Setting up company structure  │   │
│  │ ✓ Generating business trends    │   │  ← Stage checklist
│  │ ⟳ Creating June transactions... │   │     (current animates)
│  │ ○ Storing events in database    │   │
│  │ ○ Running validation            │   │
│  └─────────────────────────────────┘   │
│                                         │
│  ┌─────────────────────────────────┐   │
│  │ 💡 Did you know?                │   │  ← Educational content
│  │ Your scenario includes AC, PY,  │   │     (rotates every 10s)
│  │ and PL data following IBCS©     │   │
│  │ standards...                    │   │
│  └─────────────────────────────────┘   │
│                                         │
│           [Cancel Generation]           │  ← Cancel button
│                                         │
└─────────────────────────────────────────┘
```

---

## Component Specifications

### 1. Progress Bar

**Visual:**
- Height: 12px
- Color: Blue (#2563eb) for progress, light gray (#e5e7eb) for background
- Border radius: 6px (rounded)
- Smooth transitions (CSS: `transition: width 0.5s ease-out`)

**Behavior:**
- Never goes backwards
- Updates every 2-3 seconds based on backend `progress` field
- Minimum visible progress: 5% (so user sees something immediately)

**States:**
- Normal: Blue fill, animated shimmer effect (subtle)
- Complete: Green fill (#10b981), brief pulse animation
- Error: Red fill (#ef4444), stops animating

---

### 2. Stage Checklist

**Items (Tech Startup):**
```
✓ Setting up company structure (0-15%)
✓ Generating business trends (15-30%)
⟳ Creating [Month] transactions (30-90%, updates monthly)
○ Storing events in database (90-95%)
○ Running validation (95-100%)
```

**Icons:**
- ✓ Checkmark (green) - completed
- ⟳ Spinner (blue, animated) - current stage
- ○ Empty circle (gray) - pending

**Text:**
- Completed: Gray, line-through
- Current: Bold, black
- Pending: Light gray

**Backend Mapping (ACTUAL - as of 2025-12-10):**
```typescript
// Actual backend stage values from tier-generator.ts:
{
  'starting': 5%,
  'stage1_generating': 10%,      // Master data generation
  'stage2_generating': 30%,      // Monthly trends
  'stage3_generating': 50%,      // Start of event generation
  'stage3_2024-01_generating': 50-54%,  // January events
  'stage3_2024-02_generating': 54-58%,  // February events
  // ... (12 monthly stages, ~3.75% each)
  'stage3_2024-12_generating': ~91%,    // December events
  'database_insert': 90%,        // Storing events
  'validation': 95%,             // Running validation
  'complete': 100%               // Done
}

// Current UI only has 4 steps - need to expand to match backend
```

**RECOMMENDED: Simplified 5-Stage Display:**
```typescript
{
  'stage1': "Setting up company structure",      // 0-15%
  'stage2': "Generating business trends",        // 15-30%
  'stage3': "Creating monthly transactions",     // 30-90% (show current month)
  'database': "Storing events in database",      // 90-95%
  'validation': "Running consistency checks"     // 95-100%
}
```

---

### 3. Time Estimate

**Formula:**
```typescript
// Based on historical averages
const SCENARIO_DURATIONS = {
  bakery: 120,      // 2 minutes
  hotel: 180,       // 3 minutes
  techStartup: 450  // 7.5 minutes
}

// Calculate remaining time
const elapsed = Date.now() - startTime
const estimated = SCENARIO_DURATIONS[scenarioType]
const remaining = Math.max(0, estimated - elapsed)

// Display
if (remaining > 60) {
  return `${Math.ceil(remaining / 60)} minutes remaining`
} else {
  return `${remaining} seconds remaining`
}
```

**Display:**
- Shows estimate when progress > 5%
- Updates every 5 seconds (not every second - too distracting)
- Hides when progress > 95% (shows "Almost done!" instead)

---

### 4. Educational Content

**Purpose:** Distract user during wait, reinforce value proposition

**Content Rotation (10 seconds each):**

1. "Your scenario includes AC (Actual), PY (Previous Year), and PL (Plan) data following IBCS© standards for professional business reporting."

2. "We're generating 365 days of realistic transactions with mathematical consistency across all departments."

3. "Sales and Finance views draw from the same event stream—guaranteed data consistency."

4. "Manual creation would take 3-4 hours. We'll have it ready in ~7 minutes with AI-powered generation."

5. "Every scenario includes master data, trends, granular events, and multi-view validation."

**Visual:**
- Light blue background (#eff6ff)
- Border: 1px solid #bfdbfe
- Icon: 💡 (lightbulb)
- Font: 14px, regular
- Padding: 16px
- Fade transition between items (300ms)

---

### 5. Cancel Button

**Visual:**
- Text button (not primary color)
- Gray text (#6b7280)
- Underline on hover
- Small, unobtrusive
- Bottom center of screen

**Behavior:**
- Shows confirmation dialog (see Flow 3)
- After cancellation, returns to home but generation continues

---

### 6. Success Screen (100% Complete)

**Visual:**
```
┌─────────────────────────────────────┐
│                                     │
│         ✓ Scenario Generated!       │  ← Big checkmark (green)
│                                     │
│   Redirecting to your results...    │
│                                     │
└─────────────────────────────────────┘
```

**Timing:**
- Shows for 0.8 seconds
- Auto-redirects to `/results/[scenarioId]`
- No user action required

---

### 7. Error Screen

**Visual:**
```
┌─────────────────────────────────────┐
│                                     │
│     ⚠️ Generation Timeout           │  ← Warning icon (red)
│                                     │
│  Generation took longer than        │
│  expected. This is unusual.         │
│                                     │
│  • Usually takes 7-8 minutes        │
│  • Your data may have been created  │
│  • Check "My Scenarios" or retry    │
│                                     │
│     [Try Again]  [Back to Home]     │
│                                     │
└─────────────────────────────────────┘
```

**Button Hierarchy:**
- "Try Again" - Primary (blue button)
- "Back to Home" - Secondary (white button with border)

---

## Responsive Design

### Desktop (>768px)
- Max width: 600px centered
- Progress bar: Full width
- Generous padding and spacing

### Mobile (<768px)
- Full width with 16px margins
- Progress bar: Full width minus margins
- Slightly smaller text (14px → 13px)
- Cancel button moves to top-right corner (X icon)

---

## Accessibility

### Screen Reader Announcements

```html
<div role="status" aria-live="polite" aria-atomic="true">
  Generation 47% complete. Creating June transactions.
</div>
```

**Frequency:** Update every 10% progress or stage change

### Keyboard Navigation
- Cancel button: Tab-accessible, Enter to activate
- Confirmation dialog: Tab between buttons, Escape to dismiss

### Color Contrast
- All text meets WCAG AA standards (4.5:1 minimum)
- Progress bar distinguishable without color (use patterns/labels)

### Motion
- Respect `prefers-reduced-motion` for animations
- Provide non-animated alternative (pulsing → static)

---

## Implementation Checklist (Updated 2025-12-10)

**Completed:**
- [x] Update loading page with stage checklist ✅
- [x] Create success screen component ✅
- [x] Add screen reader announcements ✅
- [x] Basic error screen ✅
- [x] Cancel button ✅
- [x] Educational tips carousel ✅
- [x] Value reinforcement messaging ✅
- [x] Elapsed time display ✅
- [x] New tier system support ✅

**Remaining James (Dev) tasks:**
- [ ] **Task 1:** Fix stage labels to match actual backend stages
- [ ] **Task 2:** Add time estimate display (in addition to elapsed)
- [ ] **Task 3:** Add cancel confirmation dialog
- [ ] **Task 4:** Improve title display (show tier label, not raw ID)
- [ ] **Task 5:** Add "Back to Home" button on error screen
- [ ] **Task 6:** Show current month during Stage 3 (optional enhancement)

**Design assets:**
- [x] Specifications (this document)
- [x] Icon assets - using Lucide icons ✅
- [x] Color scheme - blue/gray theme ✅

---

## Future Enhancements (Post-MVP)

1. **Email Notification:** "Your scenario is ready!" email when complete
2. **Resume Generation:** If user leaves, show "In Progress" in My Scenarios
3. **Estimated Time Learning:** Improve estimates based on actual generation times
4. **A/B Test Educational Content:** Which messages keep users engaged?
5. **Scenario Preview:** Show sample data while generating (if possible)

---

## Success Metrics

**Qualitative:**
- Users understand what's happening at each stage
- Users don't feel anxious during 7-minute wait
- Error states are clear and actionable
- Professional appearance suitable for B2B demo

**Quantitative (Post-Launch):**
- <5% cancellation rate during generation
- >90% of users wait for completion
- <10% error rate perception (backend errors may happen, but UX should handle gracefully)

---

## Sally + James Alignment Session

**Required before implementation:**

1. **Review backend stages:** Ensure my stage mapping matches actual backend `currentStage` values
2. **Agree on progress calculation:** Linear vs. stage-weighted?
3. **Test time estimates:** Are historical averages accurate?
4. **Error scenarios:** What errors are possible? How should each be displayed?
5. **Cancel behavior:** Confirm backend continues after cancel

**Estimated time:** 30 minutes discussion + review

---

**Sally's Commitment:** Users will feel informed, reassured, and impressed during the wait.
