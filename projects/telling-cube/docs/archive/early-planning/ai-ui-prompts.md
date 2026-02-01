# tellingCube AI Frontend Generation Prompts

**Document Version:** 1.0
**Created:** 2025-11-14
**Purpose:** Ready-to-use prompts for AI code generation tools (v0, Lovable, etc.)

---

## Table of Contents

1. [Landing Page](#prompt-1-landing-page)
2. [Loading State](#prompt-2-loading-state)
3. [Results View](#prompt-3-results-view)
4. [Confirmation Page](#prompt-4-confirmation-page)
5. [Usage Instructions](#usage-instructions)

---

## Prompt 1: Landing Page

```markdown
# tellingCube Landing Page - AI Generation Prompt

## PROJECT CONTEXT

You are building the landing page for **tellingCube**, a B2B SaaS tool that generates realistic, multi-departmental business data in minutes using AI. The target users are business trainers, university professors, and strategy consultants who need demo datasets for workshops and presentations.

**Tech Stack:**
- Next.js 14 (App Router)
- TypeScript (strict mode)
- Tailwind CSS
- React components
- Inter font (Google Fonts)

**Brand Colors:**
- Primary: #2563EB (blue)
- Accent: #10B981 (turquoise green)
- Secondary: #E5EAF5 (light gray/white-blue)
- Text: #1F2937 (dark gray)
- Error: #EF4444 (red)

**Design Principles:**
- Clean, modern B2B SaaS aesthetic
- One clear primary action per section
- Mobile-first responsive design
- Professional credibility over flashy design

---

## HIGH-LEVEL GOAL

Create a responsive, single-page landing page (`app/page.tsx`) for tellingCube that converts visitors to users through three large scenario generation buttons (Bakery, Hotel, Tech Startup), with pricing tiers and FAQ sections below the fold.

---

## DETAILED STEP-BY-STEP INSTRUCTIONS

### Mobile Layout (< 640px):

1. **Create the file structure:**
   - Main page: `app/page.tsx`
   - Components folder: `components/landing/`
   - Create subcomponents: `Hero.tsx`, `ScenarioButtons.tsx`, `ProblemSolution.tsx`, `PricingSection.tsx`, `FAQSection.tsx`

2. **Hero Section:**
   - H1 headline: "Realistic business data in minutes, not hours" (48px, font-bold, text-center)
   - Subheadline: "Generate complete 1-year business scenarios with guaranteed multi-view consistency. No prompts. No setup. Just click." (16px, text-gray-600, text-center, max-w-2xl, mx-auto)
   - Text logo: "tellingCube.com" in top-left (Inter font, 24px, #2563EB)
   - Background: white (#FFFFFF)
   - Padding: 48px mobile, 64px desktop

3. **Scenario Buttons (Primary CTA):**
   - Three large buttons stacked vertically on mobile
   - Button text: "Generate Bakery Scenario", "Generate Hotel Scenario", "Generate Tech Startup Scenario"
   - Button style: Full-width on mobile, max-w-md, bg-blue-600 (#2563EB), text-white, py-4, px-6, rounded-lg, font-semibold, text-lg
   - Hover: bg-blue-700, scale-105 transform, transition-all duration-150
   - On click: Navigate to `/generate/loading?type=bakery` (or hotel/techStartup)
   - Space between buttons: 16px (space-y-4)

4. **Problem/Solution Section:**
   - Heading: "Stop wasting hours on demo data" (36px, font-semibold, mb-4)
   - Problem paragraph: "Business trainers, professors, and consultants spend 3-4 hours manually creating realistic datasets for each workshop. Existing tools generate inconsistent data that doesn't reconcile across departments." (text-gray-700, mb-6)
   - Solution paragraph: "tellingCube uses event-sourced AI to generate 1 year of realistic business data in under 2 minutes—with Sales, Finance, HR, and Controlling views that reconcile perfectly." (text-gray-700)
   - Background: light gray (#F9FAFB), padding 48px mobile

5. **Pricing Section (below fold):**
   - Heading: "Become a Founding Member" (36px, font-semibold, text-center, mb-2)
   - Subheading: "Lifetime access. Grandfathered pricing. Lock in your rate forever." (text-gray-600, text-center, mb-12)
   - Five pricing cards stacked vertically on mobile:
     - **Supporter S:** €29, 1 seat
     - **Supporter M:** €99, 1 seat, premium scenarios
     - **Supporter L:** €299, 5 seats
     - **Team Plus:** €599, 20 seats
     - **Department Partner:** €999, unlimited seats
   - Card structure: white background, border (#E5E7EB), rounded-lg, p-6, shadow-sm
   - Card contents: Tier name (H3, 24px, font-semibold), Price (48px, #2563EB, font-bold), Seat count (text-sm, gray), Benefits list (ul with checkmarks), CTA button ("Get [Tier Name]", bg-green-500 #10B981, full-width)
   - CTA button on click: POST to `/api/stripe/create-checkout` with tier name, then redirect to Stripe

6. **FAQ Section (accordion):**
   - Heading: "Frequently Asked Questions" (36px, font-semibold, text-center, mb-8)
   - 6 FAQ items in accordion format:
     - "What is tellingCube?" → "AI-powered tool that generates realistic business data..."
     - "How does multi-view consistency work?" → "Event-sourced architecture ensures..."
     - "Do I need an account?" → "No account needed for MVP. Just click and generate."
     - "What scenarios are available?" → "Bakery, Hotel, and Tech Startup (more coming soon)"
     - "Is my payment secure?" → "Yes, all payments processed by Stripe."
     - "Can I get a refund?" → "Yes, 30-day refund policy for founding members."
   - Accordion: Click question to expand/collapse answer, smooth transition (300ms), chevron icon rotates
   - Question style: font-semibold, text-lg, py-4, cursor-pointer, border-b
   - Answer style: text-gray-700, pb-4, hidden when collapsed

### Desktop Layout (> 1024px):

7. **Responsive adaptations:**
   - Hero: Center content, max-w-4xl
   - Scenario buttons: Display in horizontal row (flex-row, justify-center, gap-4)
   - Problem/Solution: Two-column layout optional (can keep single column centered)
   - Pricing: 5-column grid (grid-cols-5, gap-6)
   - FAQ: max-w-3xl, mx-auto

---

## CODE EXAMPLES & CONSTRAINTS

### API Route Contract (for Stripe checkout):

```typescript
// POST /api/stripe/create-checkout
// Request body:
{
  tier: "supporterS" | "supporterM" | "supporterL" | "teamPlus" | "departmentPartner"
}

// Response:
{
  checkoutUrl: string // Stripe checkout session URL
}
```

### Navigation on scenario button click:

```typescript
// Example for Bakery button
onClick={() => {
  router.push('/generate/loading?type=bakery')
}}
```

### Font loading (in layout.tsx):

```typescript
import { Inter } from 'next/font/google'
const inter = Inter({ subsets: ['latin'], display: 'swap' })
```

### Constraints:

- **DO NOT** create user authentication or login functionality
- **DO NOT** add a navigation menu (no persistent nav for MVP)
- **DO NOT** use external icon libraries beyond Heroicons (if needed)
- **DO** use semantic HTML5 (main, section, article tags)
- **DO** ensure all buttons have proper focus states (focus:ring-2 focus:ring-blue-500)
- **DO** make touch targets minimum 44x44px on mobile

---

## STRICT SCOPE

**Files to Create:**
- `app/page.tsx` (main landing page)
- `components/landing/Hero.tsx`
- `components/landing/ScenarioButtons.tsx`
- `components/landing/ProblemSolution.tsx`
- `components/landing/PricingSection.tsx`
- `components/landing/FAQSection.tsx`

**Files to Modify:**
- `tailwind.config.ts` (add custom colors if not already present)

**Files NOT to Touch:**
- Any existing API routes
- Any other pages or components
- Database configuration

**Output Requirements:**
- TypeScript with proper types for all props
- Tailwind CSS only (no custom CSS files)
- Mobile-first responsive design
- Accessibility: semantic HTML, keyboard navigation, proper ARIA labels where needed
```

---

## Prompt 2: Loading State

```markdown
# tellingCube Loading State - AI Generation Prompt

## PROJECT CONTEXT

You are building the loading/progress page for **tellingCube**, which displays while AI generates a business scenario (takes 30-90 seconds). This page must feel fast and engaging, not slow and frustrating.

**Tech Stack:**
- Next.js 14 (App Router)
- TypeScript (strict mode)
- Tailwind CSS
- React hooks (useState, useEffect)
- Inter font

**Brand Colors:**
- Primary: #2563EB (blue)
- Accent: #10B981 (turquoise green)

---

## HIGH-LEVEL GOAL

Create a full-screen loading page (`app/generate/loading/page.tsx`) that displays an animated spinner, dynamic status messages, and estimated time remaining while polling the generation API endpoint.

---

## DETAILED STEP-BY-STEP INSTRUCTIONS

### Mobile Layout (< 640px):

1. **Create the file:**
   - File: `app/generate/loading/page.tsx`
   - Use React Client Component (`'use client'`)

2. **Page Structure:**
   - Full-screen centered layout (min-h-screen, flex, items-center, justify-center)
   - Background: white
   - Content: centered column (flex-col, items-center, gap-6)

3. **Logo:**
   - "tellingCube.com" text at top (Inter font, 24px, #2563EB, mb-12)

4. **Animated Spinner:**
   - Circular spinner using SVG or Tailwind's `animate-spin`
   - Size: 64px (w-16 h-16)
   - Color: #2563EB border-4, border-t-transparent
   - Rotation: continuous (animate-spin, 1s linear infinite)

5. **Status Messages (dynamic):**
   - Display below spinner (text-lg, font-medium, text-gray-900, text-center, mb-2)
   - Messages rotate every 3 seconds:
     - "Generating 1 year of [scenario type] data..."
     - "Creating business events..."
     - "Building Sales view..."
     - "Building Finance view..."
     - "Validating consistency..."
   - Use `useState` and `useEffect` with interval to rotate messages

6. **Estimated Time Remaining:**
   - Text below status: "~45 seconds remaining" (text-sm, text-gray-600)
   - Countdown from 90 seconds to 0 using `useEffect` interval
   - Format: "~{seconds} seconds remaining"

7. **Cancel Button (optional):**
   - Text button at bottom: "Cancel" (text-blue-600, underline, text-sm, mt-8)
   - On click: Navigate back to homepage (`router.push('/')`)

8. **Generation Logic:**
   - Get scenario type from URL query param: `const searchParams = useSearchParams(); const type = searchParams.get('type')`
   - On mount, POST to `/api/generate` with `{ scenarioType: type }`
   - Poll `/api/generate/status?id={generationId}` every 2 seconds
   - When status === 'complete', redirect to `/results/{scenarioId}`
   - On error or timeout (>5 minutes), show error message with "Try Again" button

### Desktop Layout (> 1024px):

9. **Responsive adaptations:**
   - Same centered layout (no changes needed for desktop)
   - Slightly larger spinner: 80px (w-20 h-20)

---

## CODE EXAMPLES & CONSTRAINTS

### API Contract:

```typescript
// POST /api/generate
// Request:
{
  scenarioType: "bakery" | "hotel" | "techStartup"
}

// Response:
{
  generationId: string,
  status: "processing" | "complete" | "error"
}

// GET /api/generate/status?id={generationId}
// Response:
{
  status: "processing" | "complete" | "error",
  scenarioId?: string, // present when status === 'complete'
  error?: string // present when status === 'error'
}
```

### Status Message Rotation Example:

```typescript
const messages = [
  `Generating 1 year of ${scenarioType} data...`,
  "Creating business events...",
  "Building Sales view...",
  "Building Finance view...",
  "Validating consistency..."
]

const [currentMessage, setCurrentMessage] = useState(0)

useEffect(() => {
  const interval = setInterval(() => {
    setCurrentMessage((prev) => (prev + 1) % messages.length)
  }, 3000)
  return () => clearInterval(interval)
}, [])
```

### Constraints:

- **DO NOT** block the UI or prevent navigation
- **DO** handle timeout after 5 minutes with error message
- **DO** clear intervals on component unmount
- **DO** show user-friendly error messages (not technical details)
- **DO NOT** assume generation will succeed—handle errors gracefully

---

## STRICT SCOPE

**Files to Create:**
- `app/generate/loading/page.tsx`

**Files NOT to Touch:**
- API routes (assumed to exist)
- Other pages or components

**Output Requirements:**
- TypeScript with proper types
- Error handling for API failures
- Cleanup intervals on unmount
- Accessible (screen readers can announce status changes)
```

---

## Prompt 3: Results View

```markdown
# tellingCube Results View - AI Generation Prompt

## PROJECT CONTEXT

You are building the results page for **tellingCube**, which displays generated business data in two side-by-side dashboards: **Sales View** (left) and **Finance View** (right). This is the "wow moment" where users see realistic, consistent data.

**Tech Stack:**
- Next.js 14 (App Router)
- TypeScript (strict mode)
- Tailwind CSS
- Recharts (for charts)
- Inter font

**Color Palette:**

**UI Elements (buttons, tabs, badges, headers):**
- Primary: #2563EB (blue) - CTA buttons, active tabs, links
- Accent: #10B981 (green) - consistency badge, success indicators
- Error: #EF4444 (red) - error messages

**Charts & Data Visualization (IBCS-Compliant):**
- Primary data (Actual/AC): #333333 (dark gray) - main bars, lines
- Secondary data (Previous Year/PY): #999999 (medium gray) - comparison baseline
- Grid lines/axes: #CCCCCC (light gray)
- Axis labels: #666666 (medium gray)
- Positive variance (Δ >0): #27AE60 (green) - use sparingly for highlights only
- Negative variance (Δ <0): #E74C3C (red) - use sparingly for highlights only

**IMPORTANT:** IBCS colors apply ONLY to chart elements (bars, lines, data points, axes). UI elements (buttons, tabs, headers) use brand colors (#2563EB, #10B981).

---

## HIGH-LEVEL GOAL

Create a responsive, IBCS-compliant results page (`app/results/[scenarioId]/page.tsx`) that displays Sales and Finance dashboards side-by-side on desktop, with a sticky tab switcher on mobile. Each view should follow message-driven design principles with clear headlines, IBCS grayscale charts, and professional table formatting. Include a prominent consistency badge and "Become a Founding Member" CTA.

**Design Philosophy:** This is a showcase-quality IBCS dashboard demonstrating professional business communication standards. Charts use semantic grayscale colors (not decorative blues/greens), tables have no vertical lines, and every view tells a clear story.

---

## DETAILED STEP-BY-STEP INSTRUCTIONS

### Mobile Layout (< 640px):

1. **Create the file structure:**
   - Page: `app/results/[scenarioId]/page.tsx`
   - Components: `components/views/SalesView.tsx`, `components/views/FinanceView.tsx`, `components/ConsistencyBadge.tsx`
   - Charts: `components/charts/LineChart.tsx`, `components/charts/BarChart.tsx`, `components/charts/DataTable.tsx`

2. **Page Header:**
   - Logo "tellingCube.com" top-left (links to `/`)
   - Scenario metadata: "[Scenario Type] | 10 employees | 12 months" (text-sm, text-gray-600, text-center, mb-4)

3. **Consistency Badge:**
   - Display prominently below header (bg-green-50, border border-green-200, rounded-lg, p-4, mb-6, text-center)
   - Checkmark icon (green) + "✓ Multi-view consistency verified" (text-green-700, font-semibold)
   - Tooltip on hover: "Revenue, payroll, and costs reconcile perfectly across Sales and Finance views"

4. **Tab Navigation (Mobile Only):**
   - Sticky tabs at top (sticky top-0, bg-white, z-10, border-b)
   - Two tabs: "Sales" and "Finance"
   - Active tab: bg-blue-600 text-white, Inactive: text-gray-700 bg-gray-100
   - On tap, switch between SalesView and FinanceView (smooth transition 250ms)

5. **Sales View Content:**
   - **Message Headline** (IBCS "SAY" principle):
     - Example: "Bakery XYZ generated €500K revenue in 2024" (text-2xl md:text-3xl, font-semibold, mb-2, text-gray-900)
     - Optional subtitle: "Driven by strong pastry sales (+35% vs PY)" (text-base, text-gray-600, mb-6)
   - **Section Header:** "Sales View" (text-xl, font-medium, mb-4, text-gray-700)
   - **Key Metrics Cards** (3 cards in row, bg-white, border, rounded, p-4):
     - Total Revenue (AC): €500,000 (text-3xl, font-bold, text-gray-900) + label "Actual 2024"
     - Avg Monthly Revenue: €41,667 (text-xl, text-gray-700)
     - Total Units Sold: 125,000 (text-xl, text-gray-700)
   - **Revenue by Month (Line Chart - IBCS):**
     - Title above chart: "Revenue by Month" (text-lg, font-medium, mb-2)
     - X-axis: Months (Jan, Feb, Mar...) - tick color #666666
     - Y-axis: Revenue (€) - tick color #666666, start at 0
     - Grid: Horizontal only, stroke #CCCCCC, strokeDasharray "3 3"
     - Line color: #333333 (dark gray - AC/Actual), strokeWidth 2
     - Background: white, no colored fills
     - Tooltip: Show exact values with "AC 2024" label
   - **Revenue by Product (Bar Chart - IBCS):**
     - Title: "Revenue by Product Category" (text-lg, font-medium, mb-2)
     - Horizontal bars (easier to read product names)
     - Y-axis: Products (Bread, Pastries, Cakes...) - left-aligned labels, #666666
     - X-axis: Revenue (€) - tick color #666666, start at 0
     - Bar color: #333333 (dark gray - AC), no gradients
     - Grid: Vertical light lines only (#CCCCCC)
     - Sort bars by value (largest first)
   - **Top Customers (Data Table - IBCS):**
     - Title: "Top 5 Customers by Revenue" (text-lg, font-medium, mb-2)
     - Columns: Customer Name (left-align) | Revenue AC (right-align) | Orders (right-align)
     - **NO vertical lines** - use spacing only
     - Header row: border-bottom (2px, #CCCCCC), font-semibold
     - Number formatting: €125,000 (thousands separator)
     - Total row at bottom: bold, border-top (2px, #CCCCCC), bg-gray-50
     - Monospace font for numbers (font-mono) to align decimals

6. **Finance View Content:**
   - **Message Headline** (IBCS "SAY" principle):
     - Example: "Operating profit reached €75K (15% margin)" (text-2xl md:text-3xl, font-semibold, mb-2, text-gray-900)
     - Optional subtitle: "Despite 12% increase in COGS vs PY" (text-base, text-gray-600, mb-6)
   - **Section Header:** "Finance View" (text-xl, font-medium, mb-4, text-gray-700)
   - **P&L Summary Card** (bg-white, border, rounded, p-4):
     - **Table format (IBCS-compliant, no vertical lines):**
     - Row 1: Revenue | €500,000 (right-align, font-mono)
     - Row 2: Payroll | -€150,000 (right-align, text-gray-700)
     - Row 3: COGS | -€200,000 (right-align, text-gray-700)
     - Row 4: Operating Profit | €150,000 (right-align, font-bold, border-top 2px)
     - Row 5: Net Margin | 30% (right-align, text-gray-600)
   - **Monthly Cash Flow (Line Chart - IBCS):**
     - Title: "Cash Flow by Month" (text-lg, font-medium, mb-2)
     - X-axis: Months - tick color #666666
     - Y-axis: Amount (€) - tick color #666666, start at 0
     - Grid: Horizontal only, stroke #CCCCCC
     - Line 1 (Cash In): #333333 (dark gray - AC), strokeWidth 2
     - Line 2 (Cash Out): #999999 (medium gray - comparison), strokeWidth 2, strokeDasharray "5 5"
     - Legend: "Cash In (AC)" and "Cash Out" - small, top-right
     - Background: white, no colored fills
   - **Cost Breakdown (Bar Chart - IBCS):**
     - Title: "Cost Breakdown by Category" (text-lg, font-medium, mb-2)
     - Horizontal bars: Payroll, COGS, Other Expenses
     - Bar colors: All #333333 (grayscale, no decorative colors)
     - X-axis: Amount (€) - tick color #666666
     - Y-axis: Categories - left-aligned labels, #666666
     - Sort by value (largest first)
   - **Margin Analysis (Table - IBCS):**
     - Title: "Margin Analysis" (text-lg, font-medium, mb-2)
     - Columns: Metric (left-align) | Value (right-align, font-mono)
     - Rows: Gross Margin | 40%, Net Margin | 30%
     - NO vertical lines, header border-bottom only

7. **CTA Banner (bottom):**
   - Fixed at bottom on mobile (fixed bottom-0, left-0, right-0)
   - Background: #2563EB, text-white, p-4, text-center
   - Text: "Want unlimited access? Become a Founding Member"
   - Button: "View Pricing" (bg-green-500, px-6, py-2, rounded, font-semibold)
   - On click: Scroll to pricing section on homepage or navigate to `/#pricing`

8. **Data Fetching:**
   - On mount, fetch scenario data from:
     - `/api/views/sales?scenarioId={scenarioId}`
     - `/api/views/finance?scenarioId={scenarioId}`
     - `/api/validate?scenarioId={scenarioId}` (for consistency badge)
   - Show loading skeleton while fetching
   - Handle errors: "Scenario not found" or "Scenario expired (>24 hours)"

### Desktop Layout (> 1024px):

9. **Responsive adaptations:**
   - Remove tab navigation
   - Display Sales and Finance views side-by-side (grid grid-cols-2 gap-6)
   - Sales View: left column (50%)
   - Finance View: right column (50%)
   - CTA banner: not fixed, just placed at bottom of page

---

## IBCS LAYOUT & DESIGN GUIDELINES

**Reference:** See `docs/standards/ibcs-standards.md` and `docs/front-end-spec.md` for complete IBCS principles.

### Chart Types to Use (IBCS EXPRESS Principle)

**✅ Preferred:**
- **Bar/Column charts:** For comparing magnitudes (revenue by product, costs by category)
- **Line charts:** For time series (revenue by month, cash flow over time)
- **Horizontal bars:** Easier to read long category labels
- **Tables with charts:** Combine precise values with visual comparison

**❌ Avoid:**
- Pie charts, ring charts (hard to compare angles)
- Gauges, speedometers (information-poor)
- Radar charts, funnel charts (distort perception)
- 3D effects, shadows, gradients on data elements

### Clean Layout Rules (IBCS SIMPLIFY Principle)

**DO:**
- ✅ Use **white backgrounds** for charts (no colored fills behind data)
- ✅ Show **horizontal grid lines only** (light gray #CCCCCC, dashed)
- ✅ Use **minimal axes** - thin lines, no heavy borders
- ✅ Place **titles above charts** (text-lg, font-medium, mb-2)
- ✅ Use **consistent spacing** (16px/24px/32px between elements)
- ✅ **Direct labels** on data points where helpful (not just relying on legend)

**DON'T:**
- ❌ Colored backgrounds behind charts
- ❌ Heavy borders around chart containers
- ❌ Vertical grid lines (unless horizontal bars)
- ❌ Decorative elements (icons, shapes not carrying data)
- ❌ Multiple font families (stick to Inter)

### Table Formatting (IBCS Section 6.5)

**Structure Rules:**
- **NO vertical lines** - use column spacing and alignment only
- **Minimal horizontal lines** - header separator (border-bottom) and totals row (border-top) only
- **Row headers:** Left-aligned text
- **Numeric columns:** Right-aligned, monospace font (font-mono) for decimal alignment
- **Scenario abbreviations:** Use AC, PY, PL, FC (not "Actual", "Previous Year")
- **Number formatting:** Consistent decimals (€125,000 or €125K), thousands separators
- **Totals row:** Bold text, light background (bg-gray-50), border-top (2px)
- **Header row:** font-semibold, border-bottom (2px, #CCCCCC)

**Example Table Structure:**
```
Product Category    Revenue AC    Revenue PY       Δ%
─────────────────────────────────────────────────────  ← border-bottom only
Pastries               250,000       185,000     +35%
Bread                  150,000       160,000      -6%
Cakes                  100,000        90,000     +11%
─────────────────────────────────────────────────────  ← border-top for totals
Total Revenue          500,000       435,000     +15%  ← bold
```

### Color Usage (IBCS Section 6.2)

**Semantic, Not Decorative:**
- Use color to **convey meaning**, not for decoration
- Same scenario = same color everywhere (AC always #333333, PY always #999999)
- Reserve accent colors (#27AE60 green, #E74C3C red) for **highlights only** (variances, exceptions)
- Most charts should be **primarily grayscale**

**Chart Color Patterns:**
- **Single time series:** #333333 (AC) solid line
- **AC vs PY comparison:** #333333 (AC) solid, #999999 (PY) solid or dashed
- **Variance highlights:** #27AE60 (positive Δ), #E74C3C (negative Δ) - use sparingly
- **Grid/axes:** Always #CCCCCC (light gray), never bold or colored

### Visual Integrity (IBCS CHECK Principle)

**Ensure Honest Representation:**
- ✅ **Y-axis starts at 0** for bar charts (unless showing variances only)
- ✅ **Equal scales** for comparable charts (same unit = same Y-axis range)
- ✅ **Proportional bar lengths** - length must match actual value
- ❌ **No truncated axes** that exaggerate small differences
- ❌ **No logarithmic scales** without clear labeling

### Message-Driven Design (IBCS SAY Principle)

**Every View Tells a Story:**
1. **Start with the message:** Clear headline stating the key finding
   - Example: "Bakery XYZ generated €500K revenue in 2024"
2. **Add context:** Optional subtitle explaining the "why"
   - Example: "Driven by strong pastry sales (+35% vs PY)"
3. **Show evidence:** Charts and tables support the message below
4. **Keep it simple:** One main message per view, not multiple competing insights

---

## CODE EXAMPLES & CONSTRAINTS

### API Contract:

```typescript
// GET /api/views/sales?scenarioId={id}
// Response:
{
  revenueByMonth: Array<{ month: string, revenue: number }>,
  revenueByProduct: Array<{ productId: string, revenue: number, unitsSold: number }>,
  topCustomers: Array<{ name: string, totalRevenue: number, orders: number }>,
  trendAnalysis: { avgMonthlyGrowth: number }
}

// GET /api/views/finance?scenarioId={id}
// Response:
{
  plSummary: { revenue: number, payroll: number, cogs: number, profit: number },
  monthlyCashFlow: Array<{ month: string, cashIn: number, cashOut: number }>,
  costBreakdown: { payroll: number, cogs: number, other: number },
  marginAnalysis: { grossMargin: number, netMargin: number }
}

// GET /api/validate?scenarioId={id}
// Response:
{
  valid: boolean,
  errors: string[]
}
```

### Recharts Example (Line Chart - IBCS-Compliant):

```typescript
import { LineChart, Line, XAxis, YAxis, CartesianGrid, Tooltip, ResponsiveContainer } from 'recharts'

<ResponsiveContainer width="100%" height={300}>
  <LineChart data={revenueByMonth}>
    <CartesianGrid strokeDasharray="3 3" stroke="#CCCCCC" />
    <XAxis dataKey="month" tick={{ fill: '#666666' }} />
    <YAxis tick={{ fill: '#666666' }} />
    <Tooltip />
    <Line type="monotone" dataKey="revenue" stroke="#333333" strokeWidth={2} />
  </LineChart>
</ResponsiveContainer>
```

**IBCS Color Guidelines for Charts:**
- Primary data (Actual): #333333 (dark gray)
- Secondary data (Previous Year): #999999 (medium gray)
- Grid lines/axes: #CCCCCC (light gray)
- Labels: #666666 (medium gray)
- Positive variances: #27AE60 (green, use sparingly)
- Negative variances: #E74C3C (red, use sparingly)

### Constraints:

- **DO NOT** allow editing or customization of data (read-only view)
- **DO** use Recharts for all charts (not Chart.js)
- **DO** make charts responsive (use ResponsiveContainer)
- **DO** show loading states (skeleton) while fetching data
- **DO** handle expired scenarios (>24 hours) gracefully with redirect to homepage
- **DO NOT** persist any data changes (this is an ephemeral view)

---

## STRICT SCOPE

**Files to Create:**
- `app/results/[scenarioId]/page.tsx`
- `components/views/SalesView.tsx`
- `components/views/FinanceView.tsx`
- `components/ConsistencyBadge.tsx`
- `components/charts/LineChart.tsx`
- `components/charts/BarChart.tsx`
- `components/charts/DataTable.tsx`

**Dependencies to Install:**
```bash
npm install recharts
```

**Files NOT to Touch:**
- API routes
- Other pages

**Output Requirements:**
- TypeScript with proper types for all data structures
- Responsive design (mobile tabs, desktop side-by-side)
- Loading states and error handling
- Recharts for all visualizations
```

---

## Prompt 4: Confirmation Page

```markdown
# tellingCube Confirmation Page - AI Generation Prompt

## PROJECT CONTEXT

You are building the post-payment confirmation page for **tellingCube**, displayed after a user successfully completes a Stripe checkout. This page confirms their founding member status and provides next steps.

**Tech Stack:**
- Next.js 14 (App Router)
- TypeScript (strict mode)
- Tailwind CSS
- Inter font

**Brand Colors:**
- Primary: #2563EB (blue)
- Accent: #10B981 (turquoise green) - success theme

---

## HIGH-LEVEL GOAL

Create a celebratory confirmation page (`app/confirmation/page.tsx`) that confirms the payment, explains access instructions, and provides a CTA to generate the first scenario.

---

## DETAILED STEP-BY-STEP INSTRUCTIONS

### Mobile Layout (< 640px):

1. **Create the file:**
   - File: `app/confirmation/page.tsx`
   - Use React Server Component (default)

2. **Page Structure:**
   - Full-screen centered layout (min-h-screen, flex, items-center, justify-center, bg-gray-50)
   - Content card: white background, max-w-md, p-8, rounded-lg, shadow-lg, text-center

3. **Success Icon:**
   - Large green checkmark icon (w-16 h-16, text-green-500, mb-6, mx-auto)
   - Use Heroicons `CheckCircleIcon` or SVG

4. **Headline:**
   - "You're a Founding Member!" (text-3xl, font-bold, text-gray-900, mb-4)

5. **Thank You Message:**
   - "Thank you for supporting tellingCube! Your payment was successful." (text-lg, text-gray-700, mb-6)

6. **Access Instructions:**
   - Heading: "What's next?" (text-xl, font-semibold, text-gray-900, mb-3)
   - Instructions (text-gray-700, text-left, space-y-2):
     - "✓ You now have unlimited access to all scenarios"
     - "✓ No account needed—just visit tellingcube.com anytime"
     - "✓ Check your email for your receipt and confirmation"

7. **Primary CTA:**
   - Button: "Generate Your First Scenario" (bg-blue-600 #2563EB, text-white, w-full, py-3, rounded-lg, font-semibold, text-lg, mb-4)
   - On click: Navigate to `/` (homepage)
   - Hover: bg-blue-700

8. **Secondary Link:**
   - Text link: "View pricing details" (text-blue-600, underline, text-sm)
   - On click: Navigate to `/#pricing`

9. **Session Verification:**
   - Get Stripe session_id from URL query param: `const searchParams = useSearchParams(); const sessionId = searchParams.get('session_id')`
   - Verify session is valid by calling `/api/stripe/verify-session?session_id={sessionId}`
   - If invalid or missing: Show error message "Payment confirmation not found. Please check your email."

### Desktop Layout (> 1024px):

10. **Responsive adaptations:**
    - Card slightly wider: max-w-lg
    - Same content layout (centered, no major changes)

---

## CODE EXAMPLES & CONSTRAINTS

### URL Structure:

```
/confirmation?session_id=cs_test_abc123
```

### API Contract:

```typescript
// GET /api/stripe/verify-session?session_id={id}
// Response:
{
  valid: boolean,
  customerEmail?: string,
  tier?: string,
  amount?: number
}
```

### Session Verification Example:

```typescript
'use client'

import { useSearchParams } from 'next/navigation'
import { useEffect, useState } from 'react'

export default function ConfirmationPage() {
  const searchParams = useSearchParams()
  const sessionId = searchParams.get('session_id')
  const [verified, setVerified] = useState(false)
  const [error, setError] = useState(false)

  useEffect(() => {
    if (!sessionId) {
      setError(true)
      return
    }

    fetch(`/api/stripe/verify-session?session_id=${sessionId}`)
      .then(res => res.json())
      .then(data => {
        if (data.valid) {
          setVerified(true)
        } else {
          setError(true)
        }
      })
      .catch(() => setError(true))
  }, [sessionId])

  if (error) {
    return <div>Error: Payment confirmation not found.</div>
  }

  if (!verified) {
    return <div>Verifying payment...</div>
  }

  return (
    // Main confirmation UI
  )
}
```

### Constraints:

- **DO** verify the Stripe session_id is valid before showing success message
- **DO NOT** store payment details (Stripe handles this)
- **DO** show loading state while verifying session
- **DO** handle missing or invalid session_id gracefully
- **DO NOT** create account creation flow (ephemeral MVP)

---

## STRICT SCOPE

**Files to Create:**
- `app/confirmation/page.tsx`

**Files to Modify:**
- None

**Files NOT to Touch:**
- Stripe webhook handlers (separate from this page)
- API routes

**Output Requirements:**
- TypeScript with proper error handling
- Client component for session verification
- Celebratory, positive tone
- Clear next steps for user
```

---

## Usage Instructions

### How to Use These Prompts:

1. **Copy the entire prompt** (including all markdown formatting and code blocks)
2. **Paste into your AI tool:**
   - **v0.dev**: Paste directly into the chat interface
   - **Lovable.ai**: Use the "Generate Component" or "Generate Page" feature
   - **Claude/ChatGPT**: Paste into conversation for code generation
3. **Review the generated code carefully:**
   - Check TypeScript types are correct
   - Verify API contracts match your backend
   - Test responsive behavior on real devices
   - Review accessibility (keyboard navigation, screen readers)
4. **Test thoroughly:**
   - Mobile (375px width minimum)
   - Tablet (640-1023px)
   - Desktop (1024px+)
   - Error states (API failures, missing data)
   - Loading states
5. **Refine iteratively:**
   - If something's not right, provide follow-up instructions
   - Example: "Make the buttons larger on mobile" or "Change the color scheme"

---

### Prerequisites:

**Before using these prompts, ensure you have:**

1. **Next.js 14 project initialized:**
   ```bash
   npx create-next-app@latest telling-cube --typescript --tailwind --app
   ```

2. **Dependencies installed:**
   ```bash
   npm install recharts @heroicons/react
   ```

3. **Tailwind configured with brand colors** (`tailwind.config.ts`):
   ```typescript
   module.exports = {
     theme: {
       extend: {
         colors: {
           primary: '#2563EB',
           accent: '#10B981',
           secondary: '#E5EAF5',
         }
       }
     }
   }
   ```

4. **API routes implemented** (or mocked for testing)

---

### Important Reminders:

⚠️ **All AI-generated code requires:**
- Careful human review
- Testing on real devices
- Accessibility verification
- Security review (especially for payment flows)
- Performance optimization

⚠️ **API dependencies:**
- These prompts assume API routes exist at the specified endpoints
- Mock the APIs or implement them first before frontend works fully

⚠️ **Iterative approach:**
- Generate one screen at a time
- Test each screen before moving to the next
- Refine based on actual usage and feedback

---

## Related Documents

- **PRD:** `docs/prd.md` (Product requirements and user stories)
- **UI/UX Spec:** `docs/front-end-spec.md` (Detailed design specifications)
- **Project Brief:** `docs/brief.md` (Strategic context and vision)
- **Week 1 Checklist:** `docs/week-1-checklist.md` (Development plan)

---

**Document Status:** ✅ Ready for AI Code Generation

**Contact:** Sally (UX Expert)
**Version:** 1.0
**Last Updated:** 2025-11-14
