# UX Specification: Dark Mode & Sparkline Interactions
**Author:** Sally 🎨 (UX Expert)
**For:** James 💻 (Developer)
**Date:** 2025-12-12
**Status:** Ready for Implementation

---

## 1. Dark Mode

### 1.1 Overview
Add optional dark mode toggle for users who prefer reduced eye strain or work in low-light environments. This enhances accessibility and professional appearance.

### 1.2 Color Palette

| Element | Light Mode | Dark Mode |
|---------|------------|-----------|
| Background (page) | `#FFFFFF` | `#1A1A1A` |
| Background (card) | `#FFFFFF` | `#262626` |
| Background (hover) | `#F9FAFB` | `#333333` |
| Text (primary) | `#111827` | `#F3F4F6` |
| Text (secondary) | `#6B7280` | `#9CA3AF` |
| Border (card) | `#E5E7EB` | `#404040` |
| Border (divider) | `#E5E7EB` | `#333333` |
| Border (header) | `#111827` | `#D1D5DB` |

### 1.3 IBCS Chart Colors (Grayscale - Unchanged)

The IBCS grayscale palette works well in both modes:
- AC bars: `#374151` (gray-700) → Keep same
- PY bars: `#D1D5DB` (gray-300) → Keep same
- Positive variance: `#15803D` (green-700) → `#22C55E` (green-500) in dark
- Negative variance: `#B91C1C` (red-700) → `#EF4444` (red-500) in dark

### 1.4 Implementation Approach

**Option A (Recommended): CSS Variables + Tailwind**
```css
:root {
  --bg-primary: #FFFFFF;
  --bg-card: #FFFFFF;
  --text-primary: #111827;
  --text-secondary: #6B7280;
  --border-card: #E5E7EB;
}

.dark {
  --bg-primary: #1A1A1A;
  --bg-card: #262626;
  --text-primary: #F3F4F6;
  --text-secondary: #9CA3AF;
  --border-card: #404040;
}
```

**Toggle Location:** Top-right of header, using sun/moon icon toggle.

**Persistence:** Store preference in localStorage, respect `prefers-color-scheme` as default.

### 1.5 Components to Update

1. `components/results/FinanceView.tsx`
2. `components/results/SalesView.tsx`
3. `components/charts/IBCSFullPLChart.tsx`
4. `components/charts/IBCSProductChart.tsx`
5. `components/charts/IBCSRegionalChart.tsx`
6. `components/charts/IBCSCustomerMatrix.tsx`
7. `components/landing/*` (landing page components)

---

## 2. Sparkline Interactions

### 2.1 Current State
Sparklines in IBCSFullPLChart show static trend lines (revenue trend, headcount trend). No interactivity.

### 2.2 Proposed Enhancements

#### 2.2.1 Hover Tooltip
**Behavior:** On hover over sparkline, show tooltip with:
- Month label (e.g., "Mar 2024")
- Value (e.g., "€142,500")
- Change from previous month (e.g., "+5.2%")

**Design:**
```
┌─────────────────┐
│ Mar 2024        │
│ €142,500        │
│ ↑ +5.2% vs Feb  │
└─────────────────┘
```

**Tooltip styling:**
- Background: white (light) / #333 (dark)
- Border: 1px solid gray-300
- Shadow: subtle drop shadow
- Font: mono for values
- Position: Above sparkline, centered on hover point

#### 2.2.2 Hover Highlight
**Behavior:** On hover, highlight the specific data point:
- Show a small dot (4px diameter) at the hovered point
- Dot color: same as sparkline color
- Optional: Vertical line indicator

#### 2.2.3 Click to Expand (Future)
**Note:** This is a future enhancement, not for initial implementation.
- Click sparkline → Opens modal with full-size line chart
- Includes all months with detailed legend

### 2.3 Implementation Notes

**Recharts Integration:**
```tsx
<LineChart>
  <Tooltip
    content={<CustomSparklineTooltip />}
    position={{ y: -40 }}
  />
  <Line
    dot={false}
    activeDot={{ r: 4, fill: '#374151' }}
  />
</LineChart>
```

**Custom Tooltip Component:**
```tsx
interface SparklineTooltipProps {
  active?: boolean
  payload?: Array<{ value: number; payload: TrendPoint }>
  label?: string
}

function CustomSparklineTooltip({ active, payload }: SparklineTooltipProps) {
  if (!active || !payload?.length) return null
  // Render tooltip content
}
```

### 2.4 Accessibility

- Keyboard navigation: Allow Tab to focus sparkline, Enter to show tooltip
- Screen reader: Add aria-label describing the trend (e.g., "Revenue trend: increasing 12% over 12 months")
- Ensure tooltip has sufficient color contrast

---

## 3. Priority & Effort

| Feature | Priority | Estimated Complexity |
|---------|----------|---------------------|
| Sparkline hover tooltip | High | Low |
| Sparkline hover highlight | High | Low |
| Dark mode toggle | Medium | Medium |
| Dark mode chart colors | Medium | Low |
| Click to expand (future) | Low | High |

### Recommended Implementation Order:
1. Sparkline hover tooltip + highlight (quick win, improves UX immediately)
2. Dark mode toggle + CSS variables
3. Update all components for dark mode

---

## 4. Acceptance Criteria

### Sparkline Interactions
- [ ] Hovering over sparkline shows tooltip with month, value, and change %
- [ ] Tooltip appears above the sparkline, not covering the data
- [ ] Active data point is highlighted with a dot
- [ ] Tooltip disappears when mouse leaves sparkline area
- [ ] Works on both Finance and Sales view sparklines

### Dark Mode
- [ ] Toggle button in header switches between light/dark mode
- [ ] User preference persists across page refreshes
- [ ] Respects system preference on first visit
- [ ] All text remains readable in dark mode
- [ ] IBCS charts maintain professional grayscale appearance
- [ ] Variance colors (green/red) are visible in both modes
- [ ] No flash of wrong theme on page load

---

**Sally 🎨:** James, this spec should give you everything needed to implement these enhancements. The sparkline interactions are straightforward Recharts work. For dark mode, I recommend starting with CSS variables in globals.css and using Tailwind's dark: prefix. Let me know if you need clarification on any design decisions!
