# Story: "What's Next" Page Implementation

**Status:** Ready for Development
**Priority:** Medium (PoC Polish)
**Estimated Effort:** 1-2 hours
**UX Spec:** `docs/ux/whats-next-page-spec.md`

---

## Overview

Create a "What's Next" page showing the product roadmap without dates. This builds confidence for potential customers and motivates early adoption.

---

## Tasks

### Task 1: Create Feature Data Constants

**Create file:** `lib/constants/whats-next-features.ts`

```typescript
import {
  Building2,
  TrendingUp,
  PieChart,
  Download,
  Code,
  BarChart3,
  Users,
  Briefcase,
  FileSpreadsheet,
  GitCompare,
  Landmark,
  Calendar,
  Sliders,
  Database,
  Building,
  Layers,
  Link,
  FileCode,
  MessageSquare,
  LucideIcon,
} from 'lucide-react'

export type FeatureStatus = 'now' | 'next' | 'later' | 'vision'

export interface Feature {
  id: string
  title: string
  description: string
  icon: LucideIcon
  status: FeatureStatus
}

export const features: Feature[] = [
  // NOW - Currently Live
  {
    id: 'company-tiers',
    title: 'Company Tiers',
    description: 'Finance/Large Cap, Industrials/Mid Cap, Consumer Staples/Startup',
    icon: Building2,
    status: 'now',
  },
  {
    id: 'sales-view',
    title: 'Sales View',
    description: 'Revenue by month, by product, top customers, trend analysis',
    icon: TrendingUp,
    status: 'now',
  },
  {
    id: 'finance-view',
    title: 'Finance View',
    description: 'P&L summary, cash flow, cost breakdown, margin analysis',
    icon: PieChart,
    status: 'now',
  },
  {
    id: 'csv-export',
    title: 'CSV Export',
    description: 'Download any view as CSV for Excel/Power BI',
    icon: Download,
    status: 'now',
  },
  {
    id: 'rest-api',
    title: 'REST API',
    description: 'Programmatic access to scenario data',
    icon: Code,
    status: 'now',
  },
  {
    id: 'ibcs-charts',
    title: 'IBCS© Inspired Charts',
    description: 'Professional visualizations with AC, PY, PL, FC comparisons',
    icon: BarChart3,
    status: 'now',
  },

  // NEXT - Actively Building
  {
    id: 'hr-view',
    title: 'HR View',
    description: 'Headcount trends, department breakdown, payroll analysis',
    icon: Users,
    status: 'next',
  },
  {
    id: 'more-industries',
    title: 'More Industries',
    description: 'Healthcare, Retail, Professional Services sectors',
    icon: Briefcase,
    status: 'next',
  },
  {
    id: 'excel-export',
    title: 'Enhanced Exports',
    description: 'Excel (.xlsx) format, PDF reports',
    icon: FileSpreadsheet,
    status: 'next',
  },
  {
    id: 'scenario-comparison',
    title: 'Scenario Comparison',
    description: 'Compare multiple scenarios side-by-side',
    icon: GitCompare,
    status: 'next',
  },

  // LATER - On the Horizon
  {
    id: 'controlling-view',
    title: 'Controlling View',
    description: 'Cost centers, budget allocations, variance analysis',
    icon: Landmark,
    status: 'later',
  },
  {
    id: 'multi-year',
    title: 'Multi-Year Scenarios',
    description: 'Generate 3-5 year business projections',
    icon: Calendar,
    status: 'later',
  },
  {
    id: 'custom-params',
    title: 'Custom Parameters',
    description: 'Adjust growth rates, seasonality, company size',
    icon: Sliders,
    status: 'later',
  },
  {
    id: 'raw-events',
    title: 'Raw Events Export',
    description: 'Access underlying event data for custom analysis',
    icon: Database,
    status: 'later',
  },

  // VISION - Long-term Direction
  {
    id: 'enterprise',
    title: 'Enterprise Features',
    description: 'Team workspaces, role-based access, audit logs',
    icon: Building,
    status: 'vision',
  },
  {
    id: 'white-label',
    title: 'White-Label',
    description: 'Embed tellingCube in your own training platform',
    icon: Layers,
    status: 'vision',
  },
  {
    id: 'integrations',
    title: 'Integrations',
    description: 'Connect with Power BI, Tableau, SAP',
    icon: Link,
    status: 'vision',
  },
  {
    id: 'custom-templates',
    title: 'Custom Templates',
    description: 'Create industry-specific scenario templates',
    icon: FileCode,
    status: 'vision',
  },
  {
    id: 'ai-customization',
    title: 'AI Customization',
    description: 'Natural language scenario adjustments',
    icon: MessageSquare,
    status: 'vision',
  },
]

export const statusConfig: Record<FeatureStatus, {
  label: string
  badgeClass: string
  cardClass: string
  description: string
}> = {
  now: {
    label: 'Live',
    badgeClass: 'bg-green-100 text-green-700 border-green-300',
    cardClass: 'border-l-4 border-l-green-500',
    description: 'Available today',
  },
  next: {
    label: 'Building',
    badgeClass: 'bg-blue-100 text-blue-700 border-blue-300',
    cardClass: 'border-l-4 border-l-blue-500',
    description: 'In active development',
  },
  later: {
    label: 'Planned',
    badgeClass: 'bg-gray-100 text-gray-600 border-gray-300',
    cardClass: 'border-l-4 border-l-gray-400 bg-gray-50',
    description: 'Priorities may shift based on feedback',
  },
  vision: {
    label: 'Vision',
    badgeClass: 'bg-purple-100 text-purple-700 border-purple-300',
    cardClass: 'border-l-4 border-l-purple-500 bg-gradient-to-r from-purple-50 to-white',
    description: 'Our north star - where we\'re ultimately heading',
  },
}
```

---

### Task 2: Create the What's Next Page

**Create file:** `app/whats-next/page.tsx`

```tsx
import Link from 'next/link'
import { Header } from '@/components/layout/Header'
import Footer from '@/components/layout/Footer'
import { Mail } from 'lucide-react'
import { features, statusConfig, type FeatureStatus } from '@/lib/constants/whats-next-features'

export const metadata = {
  title: "What's Next | tellingCube",
  description: 'See what features are coming to tellingCube - our product roadmap and vision',
}

export default function WhatsNextPage() {
  const sections: { status: FeatureStatus; title: string }[] = [
    { status: 'now', title: 'Now' },
    { status: 'next', title: 'Next' },
    { status: 'later', title: 'Later' },
    { status: 'vision', title: 'Vision' },
  ]

  return (
    <>
      <Header />
      <main className="min-h-screen bg-gray-50">
        <div className="max-w-5xl mx-auto px-6 py-12">
          {/* Header */}
          <div className="text-center mb-12">
            <h1 className="text-3xl md:text-4xl font-bold text-gray-900 mb-4">
              What's Next
            </h1>
            <p className="text-lg text-gray-600 max-w-2xl mx-auto">
              tellingCube is actively evolving. Here's where we're headed.
              <br />
              <span className="text-blue-600 font-medium">
                Your feedback as a founding member shapes our priorities.
              </span>
            </p>
          </div>

          {/* Feature Sections */}
          {sections.map(({ status, title }) => {
            const sectionFeatures = features.filter((f) => f.status === status)
            const config = statusConfig[status]

            return (
              <section key={status} className="mb-12">
                <div className="flex items-center gap-3 mb-6">
                  <h2 className="text-2xl font-semibold text-gray-900">{title}</h2>
                  <span className={`px-3 py-1 text-xs font-semibold rounded-full border ${config.badgeClass}`}>
                    {config.label}
                  </span>
                </div>

                <p className="text-sm text-gray-500 mb-4">{config.description}</p>

                <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
                  {sectionFeatures.map((feature) => {
                    const Icon = feature.icon
                    return (
                      <div
                        key={feature.id}
                        className={`bg-white rounded-lg border border-gray-200 p-4 ${config.cardClass}`}
                      >
                        <div className="flex items-start gap-3">
                          <div className="p-2 bg-gray-100 rounded-lg shrink-0">
                            <Icon className="h-5 w-5 text-gray-600" />
                          </div>
                          <div>
                            <h3 className="font-semibold text-gray-900">{feature.title}</h3>
                            <p className="text-sm text-gray-600 mt-1">{feature.description}</p>
                          </div>
                        </div>
                      </div>
                    )
                  })}
                </div>
              </section>
            )
          })}

          {/* Feedback CTA */}
          <section className="mt-16 bg-blue-50 border border-blue-200 rounded-lg p-8 text-center">
            <div className="flex justify-center mb-4">
              <div className="p-3 bg-blue-100 rounded-full">
                <Mail className="h-6 w-6 text-blue-600" />
              </div>
            </div>
            <h2 className="text-xl font-semibold text-gray-900 mb-2">
              Shape What We Build
            </h2>
            <p className="text-gray-600 mb-4 max-w-lg mx-auto">
              As a founding member, your feedback directly influences our priorities.
              What would help you most?
            </p>
            <a
              href="mailto:support@tellingcube.com?subject=Feature%20Feedback"
              className="inline-flex items-center gap-2 px-6 py-3 bg-blue-600 text-white font-medium rounded-lg hover:bg-blue-700 transition-colors"
            >
              <Mail className="h-4 w-4" />
              Email Us Your Ideas
            </a>
          </section>

          {/* Back Link */}
          <div className="mt-12 text-center">
            <Link
              href="/"
              className="text-blue-600 hover:text-blue-700 hover:underline"
            >
              Back to Home
            </Link>
          </div>
        </div>
      </main>
      <Footer />
    </>
  )
}
```

---

### Task 3: Update Header Navigation

**File:** `components/layout/Header.tsx`

Add "What's Next" link before "My Scenarios":

```tsx
{/* Navigation */}
<nav className="flex items-center gap-6">
  <Link
    href="/whats-next"
    className="text-sm font-medium text-gray-600 hover:text-gray-900 hover:underline transition-colors"
  >
    What's Next
  </Link>
  <Link
    href="/scenarios"
    className="text-sm font-medium text-gray-600 hover:text-gray-900 hover:underline transition-colors"
  >
    My Scenarios
  </Link>
</nav>
```

---

### Task 4: Update FAQ with Links

**File:** `components/landing/FAQSection.tsx`

Update two FAQ answers:

**"What company types are available?"** - Add at end:
```
" See What's Next for upcoming industries."
```

**"Can I export the data?"** - Add at end:
```
" More export formats coming - see What's Next."
```

Note: These should be actual links. Update the FAQ structure to support JSX or append the text with a note about the page.

---

## Acceptance Criteria

- [ ] Page accessible at `/whats-next`
- [ ] Four sections: Now, Next, Later, Vision
- [ ] Each feature has icon, title, description, and status badge
- [ ] Cards styled differently per status (green/blue/gray/purple borders)
- [ ] Header navigation includes "What's Next" link
- [ ] Feedback CTA with mailto link at bottom
- [ ] Responsive design (2 columns desktop, 1 column mobile)
- [ ] FAQ mentions What's Next page (text reference is acceptable)
- [ ] Lint passes

---

## Notes

- All content is hardcoded (no CMS)
- Feature data is in a constants file for easy updates
- No dates or timelines anywhere on the page
- Uses "inspired by IBCS©" (trademark compliant)
