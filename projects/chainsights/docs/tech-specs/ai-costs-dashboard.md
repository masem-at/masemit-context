# Technical Specification: AI Costs Dashboard

**Version:** 1.0
**Date:** 2026-01-27
**Author:** Winston (Architect), with input from Amelia (Dev), Sally (UX), Mary (BA)
**Status:** Draft - Pending Approval

---

## 1. Overview

### 1.1 Purpose
Add an AI Costs Dashboard to the admin panel that provides visibility into token usage and estimated costs for all AI-powered operations (Deep Dive and Standard reports).

### 1.2 Scope

| Operation | Uses AI? | In Scope |
|-----------|----------|----------|
| Deep Dive Report | Yes | Yes |
| Standard Report | Yes | Yes |
| Reanalysis (Decline) | Yes | Yes |
| Free Health Check | No | No |
| Daily GVS Job | No | No |

### 1.3 Goals
1. **Cost Monitoring** - Track AI spending for budgeting
2. **Optimization** - Identify expensive operations
3. **Future-Proofing** - Architecture supports new AI features

---

## 2. Technical Architecture

### 2.1 Data Source

Existing `reports` table already contains token data:

```sql
-- Current schema (no changes needed)
reports:
  input_tokens    INTEGER
  output_tokens   INTEGER
  created_at      TIMESTAMP
```

**Current data available:**
- 10+ reports with token data
- Average: ~3,600 input tokens, ~1,200 output tokens per report

### 2.2 Database View (New)

Create a PostgreSQL VIEW for simplified querying:

```sql
-- Migration: 0025_add_ai_usage_view.sql
CREATE OR REPLACE VIEW ai_usage_summary AS
SELECT
  r.id,
  r.dao_name as reference_name,
  o.tier as operation_type,  -- 'standard' or 'deep_dive'
  r.input_tokens,
  r.output_tokens,
  (r.input_tokens + r.output_tokens) as total_tokens,
  -- Claude Sonnet 4 pricing: $3/1M input, $15/1M output
  ROUND(((r.input_tokens * 0.000003) + (r.output_tokens * 0.000015))::numeric, 6) as cost_usd,
  r.created_at,
  r.created_at AT TIME ZONE 'Europe/Vienna' as created_at_vienna
FROM reports r
JOIN orders o ON r.order_id = o.id
WHERE r.input_tokens IS NOT NULL
  AND r.output_tokens IS NOT NULL;
```

### 2.3 Configuration Constants

```typescript
// src/lib/config/ai-costs.ts
export const AI_COSTS_CONFIG = {
  // Claude Sonnet 4 pricing (as of Jan 2026)
  PRICING: {
    INPUT_PER_TOKEN: 0.000003,   // $3 per 1M tokens
    OUTPUT_PER_TOKEN: 0.000015,  // $15 per 1M tokens
    MODEL: 'claude-sonnet-4-20250514',
  },

  // Currency conversion (update periodically or use live API)
  EUR_USD_RATE: 0.92,  // 1 USD = 0.92 EUR

  // Display settings
  TIMEZONE: 'Europe/Vienna',
  CURRENCY: 'EUR',

  // Budget alerts (P2)
  MONTHLY_BUDGET_EUR: 100,
  WARNING_THRESHOLD: 0.8,  // 80%
}
```

---

## 3. API Design

### 3.1 GET /api/admin/costs/summary

Returns aggregated metrics for the selected period.

**Query Parameters:**
| Param | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| startDate | ISO8601 | No | Start of current month | Period start |
| endDate | ISO8601 | No | Now | Period end |
| operationType | string | No | 'all' | 'all' \| 'standard' \| 'deep_dive' |

**Response:**
```typescript
interface CostsSummaryResponse {
  period: {
    startDate: string
    endDate: string
  }
  metrics: {
    totalTokens: number
    inputTokens: number
    outputTokens: number
    totalCostEur: number
    totalCostUsd: number
    reportCount: number
    avgCostPerReport: number
  }
  breakdown: {
    standard: { count: number; tokens: number; costEur: number }
    deepDive: { count: number; tokens: number; costEur: number }
  }
  budget?: {
    monthlyLimitEur: number
    currentSpendEur: number
    percentUsed: number
    status: 'ok' | 'warning' | 'exceeded'
  }
}
```

### 3.2 GET /api/admin/costs/history

Returns time-series data for charts.

**Query Parameters:**
| Param | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| startDate | ISO8601 | No | 30 days ago | Period start |
| endDate | ISO8601 | No | Now | Period end |
| operationType | string | No | 'all' | Filter by type |
| granularity | string | No | 'day' | 'day' \| 'week' \| 'month' |

**Response:**
```typescript
interface CostsHistoryResponse {
  period: {
    startDate: string
    endDate: string
    granularity: 'day' | 'week' | 'month'
  }
  dataPoints: Array<{
    date: string  // ISO date
    standard: { count: number; tokens: number; costEur: number }
    deepDive: { count: number; tokens: number; costEur: number }
    total: { count: number; tokens: number; costEur: number }
  }>
}
```

### 3.3 GET /api/admin/costs/details

Returns paginated list of individual operations.

**Query Parameters:**
| Param | Type | Required | Default |
|-------|------|----------|---------|
| startDate | ISO8601 | No | 30 days ago |
| endDate | ISO8601 | No | Now |
| operationType | string | No | 'all' |
| page | number | No | 1 |
| limit | number | No | 20 |

**Response:**
```typescript
interface CostsDetailsResponse {
  items: Array<{
    id: string
    referenceName: string  // DAO name
    operationType: 'standard' | 'deep_dive'
    inputTokens: number
    outputTokens: number
    totalTokens: number
    costEur: number
    createdAt: string  // Vienna timezone
  }>
  pagination: {
    page: number
    limit: number
    total: number
    totalPages: number
  }
}
```

---

## 4. Frontend Architecture

### 4.1 New Files

```
src/
├── app/admin/costs/
│   └── page.tsx                    # Main costs page (Server Component)
├── components/admin/
│   ├── costs-summary-cards.tsx     # Summary metric cards
│   ├── costs-filters.tsx           # Time & operation type filters
│   ├── costs-chart-area.tsx        # Cost over time chart
│   ├── costs-chart-tokens.tsx      # Token distribution chart
│   ├── costs-table.tsx             # Detailed operations table
│   └── budget-alert-card.tsx       # Budget warning card (P2)
├── lib/
│   ├── config/ai-costs.ts          # Pricing & config constants
│   └── hooks/
│       └── use-costs-filters.ts    # Filter state management
```

### 4.2 Sidebar Navigation Update

Add new section to `AdminSection` type and `navItems`:

```typescript
// admin-sidebar.tsx
export type AdminSection = 'overview' | 'reports' | 'actions' | 'content' | 'costs'

const navItems: NavItem[] = [
  // ... existing items
  {
    id: 'costs',
    label: 'AI Costs',
    icon: DollarSign,  // from lucide-react
    description: 'Token usage and spending',
  },
]
```

### 4.3 Filter State Management

```typescript
// src/lib/hooks/use-costs-filters.ts
import { useSearchParams, useRouter } from 'next/navigation'

export type TimePeriod =
  | 'current_week'
  | 'current_month'
  | 'last_month'
  | 'current_quarter'
  | 'last_quarter'
  | 'custom'

export type OperationType = 'all' | 'standard' | 'deep_dive'

interface CostsFilters {
  timePeriod: TimePeriod
  operationType: OperationType
  customStart?: string
  customEnd?: string
}

export function useCostsFilters() {
  const searchParams = useSearchParams()
  const router = useRouter()

  // Read from URL params (enables sharing/bookmarking)
  const filters: CostsFilters = {
    timePeriod: (searchParams.get('period') as TimePeriod) || 'current_month',
    operationType: (searchParams.get('type') as OperationType) || 'all',
    customStart: searchParams.get('start') || undefined,
    customEnd: searchParams.get('end') || undefined,
  }

  // Also persist to sessionStorage for same-tab navigation
  const updateFilters = (newFilters: Partial<CostsFilters>) => {
    const params = new URLSearchParams(searchParams)
    if (newFilters.timePeriod) params.set('period', newFilters.timePeriod)
    if (newFilters.operationType) params.set('type', newFilters.operationType)
    if (newFilters.customStart) params.set('start', newFilters.customStart)
    if (newFilters.customEnd) params.set('end', newFilters.customEnd)

    sessionStorage.setItem('costsFilters', JSON.stringify({ ...filters, ...newFilters }))
    router.push(`/admin/costs?${params.toString()}`)
  }

  return { filters, updateFilters }
}
```

### 4.4 Charts Library

**Add dependency:**
```bash
npm install recharts
```

**Cost Over Time Chart:**
```typescript
// costs-chart-area.tsx
import { AreaChart, Area, XAxis, YAxis, Tooltip, ResponsiveContainer, Legend } from 'recharts'

interface DataPoint {
  date: string
  standard: number
  deepDive: number
}

export function CostsChartArea({ data }: { data: DataPoint[] }) {
  return (
    <ResponsiveContainer width="100%" height={300}>
      <AreaChart data={data}>
        <XAxis dataKey="date" />
        <YAxis tickFormatter={(v) => `€${v.toFixed(2)}`} />
        <Tooltip formatter={(v: number) => `€${v.toFixed(4)}`} />
        <Legend />
        <Area
          type="monotone"
          dataKey="standard"
          stackId="1"
          stroke="#3b82f6"
          fill="#3b82f6"
          fillOpacity={0.6}
          name="Standard"
        />
        <Area
          type="monotone"
          dataKey="deepDive"
          stackId="1"
          stroke="#8b5cf6"
          fill="#8b5cf6"
          fillOpacity={0.6}
          name="Deep Dive"
        />
      </AreaChart>
    </ResponsiveContainer>
  )
}
```

---

## 5. Timezone Handling

### 5.1 Storage
- All timestamps stored as `TIMESTAMP WITH TIME ZONE` in UTC

### 5.2 Display
- Convert to `Europe/Vienna` for all user-facing displays
- Use `date-fns-tz` for conversion:

```typescript
import { formatInTimeZone } from 'date-fns-tz'
import { AI_COSTS_CONFIG } from '@/lib/config/ai-costs'

export function formatViennaDate(date: Date | string, format = 'PPp'): string {
  return formatInTimeZone(
    new Date(date),
    AI_COSTS_CONFIG.TIMEZONE,
    format
  )
}
```

### 5.3 Date Range Calculation

```typescript
import { startOfWeek, startOfMonth, startOfQuarter, subMonths, subQuarters } from 'date-fns'
import { toZonedTime } from 'date-fns-tz'

export function getDateRange(period: TimePeriod, tz = 'Europe/Vienna') {
  const now = toZonedTime(new Date(), tz)

  switch (period) {
    case 'current_week':
      return { start: startOfWeek(now, { weekStartsOn: 1 }), end: now }
    case 'current_month':
      return { start: startOfMonth(now), end: now }
    case 'last_month':
      const lastMonth = subMonths(now, 1)
      return { start: startOfMonth(lastMonth), end: startOfMonth(now) }
    case 'current_quarter':
      return { start: startOfQuarter(now), end: now }
    case 'last_quarter':
      const lastQuarter = subQuarters(now, 1)
      return { start: startOfQuarter(lastQuarter), end: startOfQuarter(now) }
    default:
      return { start: startOfMonth(now), end: now }
  }
}
```

---

## 6. Dependencies

### 6.1 New NPM Packages

```json
{
  "recharts": "^2.12.0",
  "date-fns-tz": "^3.1.0"
}
```

### 6.2 Database Migration

Single migration file: `0025_add_ai_usage_view.sql`

---

## 7. Implementation Stories

### Story 1: Database Layer (P0)
**AC:**
- [ ] Create `ai_usage_summary` VIEW
- [ ] Create `src/lib/config/ai-costs.ts` with pricing constants
- [ ] Add `date-fns-tz` dependency
- [ ] Write utility functions for date/timezone handling

**Files:** `schema.ts`, `ai-costs.ts`, migration file

### Story 2: API Endpoints (P0)
**AC:**
- [ ] Implement `GET /api/admin/costs/summary`
- [ ] Implement `GET /api/admin/costs/history`
- [ ] Implement `GET /api/admin/costs/details`
- [ ] All endpoints require admin auth
- [ ] All endpoints support filter params

**Files:** 3 route files in `src/app/api/admin/costs/`

### Story 3: Costs Page - Core (P0)
**AC:**
- [ ] Create `/admin/costs` page
- [ ] Add "AI Costs" to sidebar navigation
- [ ] Implement 4 summary cards (Tokens, Cost, Avg, Count)
- [ ] Implement time period filter dropdown
- [ ] Implement operation type filter dropdown

**Files:** `page.tsx`, `costs-summary-cards.tsx`, `costs-filters.tsx`, `admin-sidebar.tsx`

### Story 4: Charts (P0)
**AC:**
- [ ] Add `recharts` dependency
- [ ] Implement stacked area chart (Cost Over Time)
- [ ] Implement horizontal bar chart (Token Distribution)
- [ ] Charts respond to filter changes

**Files:** `costs-chart-area.tsx`, `costs-chart-tokens.tsx`

### Story 5: Filter Persistence (P1)
**AC:**
- [ ] Filters persist in URL params
- [ ] Filters restore from sessionStorage on page load
- [ ] Filters sync between URL and sessionStorage

**Files:** `use-costs-filters.ts`

### Story 6: Report Cards Enhancement (P0)
**AC:**
- [ ] Add token count to report detail view
- [ ] Add estimated cost (EUR) to report detail view
- [ ] Format timestamps in Vienna timezone

**Files:** Existing report detail component

### Story 7: Budget Alerts (P2)
**AC:**
- [ ] Add budget threshold to config
- [ ] Create budget alert card component
- [ ] Show warning at 80%, danger at 100%
- [ ] Display on costs page when threshold approached

**Files:** `budget-alert-card.tsx`, `ai-costs.ts`

---

## 8. Testing Plan

### 8.1 Unit Tests
- Date range calculation functions
- Cost calculation functions
- Filter state management hook

### 8.2 Integration Tests
- API endpoints return correct data structure
- Filters correctly modify API queries

### 8.3 Manual Testing
- Verify Vienna timezone displays correctly
- Verify charts render with real data
- Test all filter combinations

---

## 9. Rollout Plan

1. **Phase 1 (MVP):** Stories 1-4, 6 — Core functionality
2. **Phase 2:** Story 5 — Filter persistence
3. **Phase 3:** Story 7 — Budget alerts

---

## 10. Open Questions

1. ~~Do we need live EUR/USD conversion or is static rate acceptable?~~ **Resolved: Static rate, configurable**
2. Should we add export to CSV functionality? **Deferred to future enhancement**
3. Do we want email alerts when budget threshold exceeded? **Deferred to P2**

---

## Approval

| Role | Name | Approved | Date |
|------|------|----------|------|
| Product Manager | John | [ ] | |
| Architect | Winston | [x] | 2026-01-27 |
| Developer | Amelia | [ ] | |
| UX Designer | Sally | [ ] | |
| Stakeholder | Mario | [ ] | |
