# Story: Global Header Implementation

**Status:** Ready for Development
**Priority:** High (Navigation is broken on secondary pages)
**Estimated Effort:** 30-45 minutes
**UX Spec:** `docs/ux/global-header-spec.md`

---

## Overview

Create a simple, consistent header for all pages. Currently, pages like Privacy and Imprint have no navigation - users are stuck without browser back button.

---

## Tasks

### Task 1: Create Header Component

**Create file:** `components/layout/Header.tsx`

```tsx
'use client'

import Link from 'next/link'
import { Box } from 'lucide-react'

export function Header() {
  return (
    <header className="sticky top-0 z-50 w-full border-b border-gray-200 bg-white">
      <div className="max-w-7xl mx-auto px-6 h-16 flex items-center justify-between">
        {/* Logo - links to home */}
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

export default Header
```

---

### Task 2: Add Header to Landing Page

**File:** `app/page.tsx`

Add import and component at the top of the page content:

```tsx
import { Header } from '@/components/layout/Header'

export default function Home() {
  return (
    <>
      <Header />
      {/* existing content */}
    </>
  )
}
```

**Note:** May need to adjust existing hero section if it has its own logo.

---

### Task 3: Add Header to My Scenarios Page

**File:** `app/scenarios/page.tsx`

Same pattern - add Header at top.

---

### Task 4: Add Header to Results Page

**File:** `app/results/[scenarioId]/page.tsx`

Same pattern - add Header at top.

---

### Task 5: Add Header to Privacy Page

**File:** `app/privacy/page.tsx`

Same pattern - this page currently has NO navigation!

---

### Task 6: Add Header to Imprint Page

**File:** `app/imprint/page.tsx`

Same pattern - this page currently has NO navigation!

---

### Task 7: Skip Loading Page

**File:** `app/generate/loading/page.tsx`

Do NOT add header here - keep users focused on the generation progress.

---

### Task 8: Clean Up Duplicate Logos

After adding the global header, check if any pages have duplicate logo elements that should be removed:
- Landing page hero might have its own logo
- Results page might have header logo

Remove duplicates to keep it clean.

---

## Acceptance Criteria

- [ ] Header component created in `components/layout/`
- [ ] Header visible on: Landing, My Scenarios, Results, Privacy, Imprint
- [ ] Header NOT on: Loading page
- [ ] Logo links to home (`/`)
- [ ] "My Scenarios" links to `/scenarios`
- [ ] Header is sticky (stays on scroll)
- [ ] No duplicate logos on any page
- [ ] Looks good on mobile
- [ ] Lint passes

---

## Design Reference

```
┌─────────────────────────────────────────────────────────────────────────┐
│   [□] tellingCube                                      My Scenarios →   │
└─────────────────────────────────────────────────────────────────────────┘
```

- Height: 64px
- Background: White with bottom border
- Sticky position
- Max-width: 1280px centered

Full spec: `docs/ux/global-header-spec.md`
