# Icon System Documentation

**Version:** 1.0
**Last Updated:** 2025-11-19
**Library:** [lucide-react](https://lucide.dev/) v0.553.0

---

## Overview

tellingCube uses a consistent icon system based on lucide-react to provide visual hierarchy, improve usability, and reinforce brand identity across the application.

---

## Icon Library

**Technology:** lucide-react
**Installation:** Already included in dependencies
**Import:** `import { IconName } from 'lucide-react'`

### Why lucide-react?

- ✅ **Consistent Design Language** - All icons follow the same visual style
- ✅ **Tree-shakeable** - Only imported icons are included in bundle
- ✅ **TypeScript Support** - Full type safety
- ✅ **Accessibility** - Proper ARIA attributes built-in
- ✅ **Customizable** - Size, color, stroke width easily adjustable

---

## Icon Size Standards

Use these standard Tailwind classes for consistent sizing:

| Size | Class | Pixels | Use Case |
|------|-------|--------|----------|
| **sm** | `h-4 w-4` | 16px | Inline with text, small UI elements |
| **md** | `h-5 w-5` | 20px | Default for buttons, navigation |
| **lg** | `h-6 w-6` | 24px | Scenario cards, feature icons |
| **xl** | `h-8 w-8` | 32px | Section headers, primary features |
| **2xl** | `h-12 w-12` | 48px | Hero elements, large callouts |

### Examples

```tsx
// Small - Inline with text
<Check className="h-4 w-4 text-green-500" />

// Medium - Navigation/Buttons
<ChevronRight className="h-5 w-5" />

// Large - Feature icons
<Store className="h-6 w-6 text-blue-600" />

// Extra Large - Section headers
<Sparkles className="h-8 w-8 text-blue-600" />
```

---

## Brand Colors for Icons

Match icons to brand color palette for semantic meaning:

| Color | Class | Hex | Usage |
|-------|-------|-----|-------|
| **Primary Blue** | `text-blue-600` | #2563EB | Solution, features, primary actions |
| **Success Green** | `text-green-500` | #10B981 | Success states, confirmation, positive |
| **Error Red** | `text-red-600` | #DC2626 | Errors, problems, warnings |
| **Warning Yellow** | `text-yellow-500` | #EAB308 | Caution, alerts |
| **Neutral Gray** | `text-gray-600` | #4B5563 | Secondary, informational |
| **Dark Gray** | `text-gray-900` | #111827 | High emphasis, headings |

### Color Usage Guidelines

- **Blue** - Innovation, solutions, primary features
- **Green** - Success, completion, positive outcomes
- **Red** - Problems, errors, critical issues
- **Gray** - Neutral information, secondary actions

---

## Icon Categories

### Status & Feedback Icons

| Icon | Name | Use Case | Color |
|------|------|----------|-------|
| ✓ | `CheckCircle2` | Success confirmation | `text-green-500` |
| ✓ | `Check` | List items, features included | `text-green-500` |
| ✗ | `XCircle` | Error states | `text-red-600` |
| ⚠ | `AlertCircle` | Problems, warnings | `text-red-600` |
| ⚠ | `AlertTriangle` | Caution | `text-yellow-500` |
| ⟳ | `Loader2` | Loading states | `text-blue-600` |

**Example:**
```tsx
import { CheckCircle2, XCircle, Loader2 } from 'lucide-react'

// Success state
<CheckCircle2 className="h-16 w-16 text-green-600" />

// Error state
<XCircle className="h-16 w-16 text-red-600" />

// Loading state
<Loader2 className="h-12 w-12 text-blue-600 animate-spin" />
```

### Scenario Icons

| Icon | Name | Scenario | Color |
|------|------|----------|-------|
| 🏪 | `Store` | Bakery | Brand color |
| 🏨 | `Hotel` | Hotel | Brand color |
| 🚀 | `Rocket` | Tech Startup | Brand color |

**Example:**
```tsx
import { Store, Hotel, Rocket } from 'lucide-react'

const scenarioIcons = {
  bakery: Store,
  hotel: Hotel,
  techStartup: Rocket,
}
```

### Navigation Icons

| Icon | Name | Use Case |
|------|------|----------|
| → | `ChevronRight` | Next, forward, expand |
| ↓ | `ChevronDown` | Dropdown, collapse |
| ↑ | `ChevronUp` | Collapse, scroll up |
| ↗ | `ExternalLink` | External links |

### Feature & Benefit Icons

| Icon | Name | Use Case | Color |
|------|------|----------|-------|
| ✨ | `Sparkles` | Solutions, AI, innovation | `text-blue-600` |
| ⚡ | `Zap` | Speed, power | `text-blue-600` |
| 💡 | `Lightbulb` | Ideas, insights | `text-blue-600` |
| 📊 | `BarChart3` | Data, analytics | `text-gray-600` |
| 🎯 | `Target` | Accuracy, precision | `text-blue-600` |

---

## Usage Guidelines

### 1. Always Pair Icons with Text

**❌ Don't:**
```tsx
<button>
  <ChevronRight />
</button>
```

**✅ Do:**
```tsx
<button className="flex items-center gap-2">
  <span>Next</span>
  <ChevronRight className="h-5 w-5" />
</button>
```

**Exception:** Universally recognized icons (X for close, hamburger menu)

### 2. Consistent Sizing in Context

Use the same icon size for similar UI elements:

```tsx
// All scenario cards use h-6 w-6
<Store className="h-6 w-6" />
<Hotel className="h-6 w-6" />
<Rocket className="h-6 w-6" />

// All status icons use h-8 w-8
<CheckCircle2 className="h-8 w-8 text-green-600" />
<XCircle className="h-8 w-8 text-red-600" />
```

### 3. Use shrink-0 to Prevent Icon Squashing

```tsx
<div className="flex items-center gap-3">
  <AlertCircle className="h-8 w-8 text-red-600 shrink-0" />
  <h2>Very Long Heading That Might Wrap</h2>
</div>
```

### 4. Semantic Color Usage

Match icon color to its semantic meaning:

```tsx
// Success - Green
<CheckCircle2 className="h-8 w-8 text-green-600" />

// Error - Red
<AlertCircle className="h-8 w-8 text-red-600" />

// Primary Feature - Blue
<Sparkles className="h-8 w-8 text-blue-600" />
```

### 5. Accessible Icon Usage

Icons should enhance, not replace, text:

```tsx
// Good - Icon + Text
<div className="flex items-center gap-2">
  <Store className="h-6 w-6" />
  <span>Bakery Scenario</span>
</div>

// Also Good - For screen readers
<button aria-label="Close dialog">
  <XCircle className="h-5 w-5" />
</button>
```

---

## Current Implementation

### Landing Page

**Hero Section**
- Currently text-only (icons optional for future)

**Problem/Solution Section** ✅
```tsx
// Problem
<AlertCircle className="h-8 w-8 text-red-600 shrink-0" />

// Solution
<Sparkles className="h-8 w-8 text-blue-600 shrink-0" />
```

**Scenario Buttons** ✅
```tsx
// Bakery
<Store className="h-6 w-6 shrink-0" />

// Hotel
<Hotel className="h-6 w-6 shrink-0" />

// Tech Startup
<Rocket className="h-6 w-6 shrink-0" />
```

**Pricing Section** ✅
```tsx
// Feature checkmarks
<Check className="h-4 w-4 text-green-500 shrink-0 mt-0.5" />
<Check className="h-5 w-5 text-green-500 shrink-0 mt-0.5" />
```

### Payment Flow

**Confirmation Page** ✅
```tsx
// Loading
<Loader2 className="h-12 w-12 text-blue-600 animate-spin" />

// Success
<CheckCircle2 className="h-16 w-16 text-green-600" />

// Error
<XCircle className="h-16 w-16 text-red-600" />
```

### Other Components

**Accordion** ✅
```tsx
<ChevronDown className="h-4 w-4 transition-transform" />
```

**Consistency Badge** ✅
```tsx
<CheckCircle className="h-4 w-4 text-green-500" />
<AlertTriangle className="h-4 w-4 text-yellow-500" />
```

---

## Code Examples

### Section Header with Icon

```tsx
import { Sparkles } from 'lucide-react'

<div className="flex items-center gap-3 mb-4">
  <Sparkles className="h-8 w-8 text-blue-600 shrink-0" />
  <h2 className="text-2xl font-bold text-gray-900">The Solution</h2>
</div>
```

### Button with Icon

```tsx
import { ChevronRight } from 'lucide-react'

<button className="flex items-center gap-2">
  <span>Continue</span>
  <ChevronRight className="h-5 w-5" />
</button>
```

### Feature List with Icons

```tsx
import { Check } from 'lucide-react'

<ul className="space-y-2">
  {features.map((feature) => (
    <li key={feature} className="flex items-start gap-2">
      <Check className="h-5 w-5 text-green-500 shrink-0 mt-0.5" />
      <span>{feature}</span>
    </li>
  ))}
</ul>
```

### Dynamic Icon Component

```tsx
import type { LucideIcon } from 'lucide-react'

interface FeatureProps {
  icon: LucideIcon
  title: string
}

const Feature = ({ icon: Icon, title }: FeatureProps) => (
  <div className="flex items-center gap-3">
    <Icon className="h-6 w-6 text-blue-600" />
    <h3>{title}</h3>
  </div>
)
```

---

## Best Practices Summary

1. ✅ **Always pair icons with text** (except universal icons)
2. ✅ **Use consistent sizing** within the same context
3. ✅ **Apply semantic colors** (green=success, red=error, blue=primary)
4. ✅ **Add shrink-0** to prevent icon squashing in flex layouts
5. ✅ **Match icon metaphors** to their purpose (Sparkles=innovation, AlertCircle=problem)
6. ✅ **Import only what you need** for optimal bundle size
7. ✅ **Use TypeScript types** (LucideIcon) for type safety
8. ❌ **Don't use random icon sizes** - stick to the standard scale
9. ❌ **Don't overuse icons** - they should enhance, not clutter
10. ❌ **Don't use icons without accessibility** in mind

---

## Future Enhancements

### Potential Additions

- **Hero Brand Icon** - Consider adding a cube or data icon to hero section
- **Feature Highlights** - If feature section added, use appropriate icons
- **Footer Icons** - Social media, contact icons
- **Dashboard Icons** - Finance, Sales, HR view icons

### Icon Animation

lucide-react icons can be animated with Tailwind:

```tsx
// Spin animation for loading
<Loader2 className="h-6 w-6 animate-spin" />

// Pulse animation for attention
<AlertCircle className="h-6 w-6 animate-pulse text-red-600" />

// Hover effects
<ChevronRight className="h-5 w-5 group-hover:translate-x-1 transition-transform" />
```

---

## Resources

- **lucide-react Documentation:** https://lucide.dev/guide/packages/lucide-react
- **Icon Search:** https://lucide.dev/icons/
- **Tailwind Colors:** https://tailwindcss.com/docs/customizing-colors
- **ARIA Labels:** https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Attributes/aria-label

---

## Maintenance

**When Adding New Icons:**
1. Check if icon exists in lucide-react library
2. Follow size guidelines (sm/md/lg/xl/2xl)
3. Apply semantic brand colors
4. Pair with text labels
5. Test on mobile and desktop
6. Update this documentation

**Version History:**
- v1.0 (2025-11-19) - Initial icon system documentation (Story 3.6a)
