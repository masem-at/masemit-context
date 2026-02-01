# Global Header UX Specification

**Author:** Sally (UX Expert)
**Date:** 2025-12-10
**Status:** Ready for Implementation

---

## Problem Statement

Users navigating to secondary pages (Privacy, Imprint, Results) have no way to navigate back except browser back button. This is unprofessional and frustrating.

**Current Issues:**
- No consistent header across pages
- Privacy/Imprint pages are dead ends
- Brand presence inconsistent
- No quick access to "My Scenarios"

---

## Design Goals

1. **Simple** - Minimal for PoC, not overengineered
2. **Consistent** - Same header on every page
3. **Functional** - Logo links home, one nav link
4. **Professional** - Clean B2B aesthetic

---

## Header Design

### Desktop (≥768px)

```
┌─────────────────────────────────────────────────────────────────────────┐
│                                                                         │
│   [Cube Icon] tellingCube                              My Scenarios →   │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
    ↑                                                          ↑
    Links to home                                    Links to /scenarios
```

### Mobile (<768px)

```
┌─────────────────────────────────────────────────────────────────────────┐
│                                                                         │
│   [Cube Icon] tellingCube                              My Scenarios →   │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

Same layout - no hamburger menu needed for PoC (only one nav item).

---

## Visual Specifications

### Container
- **Height:** 64px
- **Background:** White (`#FFFFFF`)
- **Border:** 1px solid gray-200 (`#E5E7EB`) on bottom
- **Position:** Sticky top (stays visible on scroll)
- **Z-index:** 50 (above content, below modals)
- **Padding:** 0 24px (horizontal)
- **Max-width:** 1280px centered (matches landing page)

### Logo Section (Left)
- **Icon:** Lucide `Box` or `Boxes` icon (cube representation)
- **Icon size:** 28px
- **Icon color:** Blue-600 (`#2563EB`)
- **Text:** "tellingCube"
- **Text size:** 20px, font-semibold
- **Text color:** Gray-900 (`#111827`)
- **Gap:** 8px between icon and text
- **Hover:** Opacity 80%, cursor pointer
- **Link:** Navigates to `/` (home)

### Navigation (Right)
- **Text:** "My Scenarios"
- **Text size:** 14px, font-medium
- **Text color:** Gray-600 (`#4B5563`)
- **Hover:** Gray-900, underline
- **Arrow:** Small chevron-right icon (optional, subtle)
- **Link:** Navigates to `/scenarios`

---

## Component Structure

```
components/
  layout/
    Header.tsx        ← New component
    Footer.tsx        ← Existing (if any)
```

### Header.tsx

```tsx
'use client'

import Link from 'next/link'
import { Box } from 'lucide-react'

export function Header() {
  return (
    <header className="sticky top-0 z-50 w-full border-b border-gray-200 bg-white">
      <div className="max-w-7xl mx-auto px-6 h-16 flex items-center justify-between">
        {/* Logo */}
        <Link
          href="/"
          className="flex items-center gap-2 hover:opacity-80 transition-opacity"
        >
          <Box className="h-7 w-7 text-blue-600" />
          <span className="text-xl font-semibold text-gray-900">
            tellingCube
          </span>
        </Link>

        {/* Navigation */}
        <nav>
          <Link
            href="/scenarios"
            className="text-sm font-medium text-gray-600 hover:text-gray-900 hover:underline transition-colors"
          >
            My Scenarios
          </Link>
        </nav>
      </div>
    </header>
  )
}
```

---

## Page Integration

### Pages that need Header:

| Page | Path | Notes |
|------|------|-------|
| Landing | `/` | Add header above hero |
| My Scenarios | `/scenarios` | Add header |
| Results | `/results/[id]` | Add header |
| Privacy | `/privacy` | Add header (currently no nav!) |
| Imprint | `/imprint` | Add header (currently no nav!) |
| Loading | `/generate/loading` | **Skip** - keep focused on progress |

### Layout Options

**Option A: Add to each page individually**
- Simple, explicit
- More repetition

**Option B: Create layout wrapper** (Recommended)
- Create `app/layout-with-header.tsx` or modify root layout
- Cleaner, DRY

**Recommendation:** For PoC, add to each page individually. Keeps it simple and allows exceptions (loading page).

---

## Behavior

### Logo Click
- Always navigates to `/` (home)
- No confirmation needed

### "My Scenarios" Click
- Navigates to `/scenarios`
- Shows list of generated scenarios

### Scroll Behavior
- Header stays fixed at top (sticky)
- Content scrolls underneath
- Subtle shadow appears on scroll (optional enhancement)

---

## Accessibility

- Logo link has descriptive text (not just icon)
- Navigation links are keyboard accessible
- Sufficient color contrast (gray-600 on white = 5.7:1 ✓)
- Focus states visible

---

## Future Enhancements (Not for PoC)

- User account menu (Login/Logout)
- Dropdown with more nav items
- Search functionality
- Notification bell
- Dark mode toggle

---

## Implementation Checklist

**James (Dev) tasks:**

### Task 1: Create Header Component
- [ ] Create `components/layout/Header.tsx`
- [ ] Implement logo with link to home
- [ ] Implement "My Scenarios" nav link
- [ ] Style per specifications above

### Task 2: Add Header to Landing Page
- [ ] Import Header in `app/page.tsx`
- [ ] Add above existing content
- [ ] Verify layout doesn't break

### Task 3: Add Header to My Scenarios Page
- [ ] Import Header in `app/scenarios/page.tsx`
- [ ] Add at top of page

### Task 4: Add Header to Results Page
- [ ] Import Header in `app/results/[scenarioId]/page.tsx`
- [ ] Add at top of page

### Task 5: Add Header to Privacy Page
- [ ] Import Header in `app/privacy/page.tsx`
- [ ] Add at top of page

### Task 6: Add Header to Imprint Page
- [ ] Import Header in `app/imprint/page.tsx`
- [ ] Add at top of page

### Task 7: Verify & Test
- [ ] Test all navigation links work
- [ ] Test responsive layout (mobile)
- [ ] Verify sticky behavior on scroll
- [ ] Run lint

---

## Acceptance Criteria

- [ ] Header visible on all pages except loading page
- [ ] Logo links to home from any page
- [ ] "My Scenarios" links to scenarios list
- [ ] Header stays fixed on scroll
- [ ] Looks professional and consistent
- [ ] Works on mobile widths

---

**Sally's Note:** Keep it simple! One component, consistent everywhere. We can enhance later.
