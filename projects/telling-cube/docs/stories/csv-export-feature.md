# Story: CSV Export for Aggregated Views

**Status:** Ready for Development
**Priority:** Medium (PoC Polish)
**Estimated Effort:** 1-2 hours

---

## Overview

Add CSV download buttons to the Results page, allowing users to export aggregated Finance and Sales data. This demonstrates real-world value - finance professionals expect to be able to load data into their own tools (Excel, Power BI, etc.).

---

## User Story

As a finance professional viewing generated scenario data,
I want to download the aggregated views as CSV files,
So that I can analyze the data in my own tools (Excel, Power BI, Tableau).

---

## Scope

### In Scope
- CSV export button for Finance Summary (monthly P&L)
- CSV export button for Sales Summary (monthly sales by product)
- Client-side CSV generation (no new API endpoints needed)

### Out of Scope (Future)
- JSON export
- Raw events export
- API endpoints for programmatic access
- Excel (.xlsx) format

---

## Tasks

### Task 1: Create CSV Utility Function

**Create file:** `lib/utils/csv-export.ts`

```typescript
/**
 * CSV Export Utilities
 *
 * Converts data arrays to CSV format and triggers download.
 */

/**
 * Convert array of objects to CSV string
 */
export function arrayToCSV<T extends Record<string, unknown>>(
  data: T[],
  columns?: { key: keyof T; header: string }[]
): string {
  if (data.length === 0) return ''

  // Use provided columns or derive from first object
  const cols = columns || Object.keys(data[0]).map(key => ({
    key: key as keyof T,
    header: key as string
  }))

  // Header row
  const header = cols.map(col => `"${col.header}"`).join(',')

  // Data rows
  const rows = data.map(row =>
    cols.map(col => {
      const value = row[col.key]
      if (value === null || value === undefined) return ''
      if (typeof value === 'number') return value.toString()
      return `"${String(value).replace(/"/g, '""')}"`
    }).join(',')
  )

  return [header, ...rows].join('\n')
}

/**
 * Trigger CSV file download in browser
 */
export function downloadCSV(csvContent: string, filename: string): void {
  const blob = new Blob([csvContent], { type: 'text/csv;charset=utf-8;' })
  const link = document.createElement('a')
  const url = URL.createObjectURL(blob)

  link.setAttribute('href', url)
  link.setAttribute('download', filename)
  link.style.visibility = 'hidden'

  document.body.appendChild(link)
  link.click()
  document.body.removeChild(link)

  URL.revokeObjectURL(url)
}
```

---

### Task 2: Add Export Button to Finance View

**File:** `components/results/FinanceView.tsx`

**Add import:**
```typescript
import { Download } from 'lucide-react'
import { arrayToCSV, downloadCSV } from '@/lib/utils/csv-export'
```

**Add export handler:**
```typescript
const handleExportCSV = () => {
  if (!data?.monthlyCashFlow) return

  const csvData = data.monthlyCashFlow.map(row => ({
    month: row.month,
    revenue: row.revenue,
    cogs: row.cogs,
    grossProfit: row.grossProfit,
    opex: row.opex,
    ebitda: row.ebitda,
    // Include comparison data
    revenuePY: row.previousYear,
    revenuePL: row.plan,
    revenueFC: row.forecast
  }))

  const columns = [
    { key: 'month', header: 'Month' },
    { key: 'revenue', header: 'Revenue (AC)' },
    { key: 'revenuePY', header: 'Revenue (PY)' },
    { key: 'revenuePL', header: 'Revenue (PL)' },
    { key: 'revenueFC', header: 'Revenue (FC)' },
    { key: 'cogs', header: 'COGS' },
    { key: 'grossProfit', header: 'Gross Profit' },
    { key: 'opex', header: 'OPEX' },
    { key: 'ebitda', header: 'EBITDA' }
  ]

  const csv = arrayToCSV(csvData, columns)
  downloadCSV(csv, `finance-summary-${scenarioId}.csv`)
}
```

**Add button to header (near the view title):**
```tsx
<div className="flex items-center justify-between mb-4">
  <h3 className="text-lg font-semibold text-gray-900">
    Monthly Cash Flow: AC vs PY vs PL vs FC
  </h3>
  <button
    onClick={handleExportCSV}
    className="flex items-center gap-2 px-3 py-1.5 text-sm text-gray-600 hover:text-gray-900 hover:bg-gray-100 rounded-md transition-colors"
    title="Download as CSV"
  >
    <Download className="h-4 w-4" />
    <span>CSV</span>
  </button>
</div>
```

---

### Task 3: Add Export Button to Sales View

**File:** `components/results/SalesView.tsx`

**Add import:**
```typescript
import { Download } from 'lucide-react'
import { arrayToCSV, downloadCSV } from '@/lib/utils/csv-export'
```

**Add export handler:**
```typescript
const handleExportCSV = () => {
  if (!data?.revenueByMonth) return

  const csvData = data.revenueByMonth.map(row => ({
    month: row.month,
    revenue: row.revenue,
    previousYear: row.previousYear,
    plan: row.plan,
    forecast: row.forecast,
    variancePY: row.revenue - (row.previousYear || 0),
    variancePYPercent: row.previousYear
      ? ((row.revenue - row.previousYear) / row.previousYear * 100).toFixed(1)
      : 'N/A'
  }))

  const columns = [
    { key: 'month', header: 'Month' },
    { key: 'revenue', header: 'Revenue (AC)' },
    { key: 'previousYear', header: 'Revenue (PY)' },
    { key: 'plan', header: 'Revenue (PL)' },
    { key: 'forecast', header: 'Revenue (FC)' },
    { key: 'variancePY', header: 'Variance PY (€)' },
    { key: 'variancePYPercent', header: 'Variance PY (%)' }
  ]

  const csv = arrayToCSV(csvData, columns)
  downloadCSV(csv, `sales-summary-${scenarioId}.csv`)
}
```

**Add button to header:**
```tsx
<div className="flex items-center justify-between mb-4">
  <h3 className="text-lg font-semibold text-gray-900">
    Revenue Trend: AC vs PY vs PL vs FC
  </h3>
  <button
    onClick={handleExportCSV}
    className="flex items-center gap-2 px-3 py-1.5 text-sm text-gray-600 hover:text-gray-900 hover:bg-gray-100 rounded-md transition-colors"
    title="Download as CSV"
  >
    <Download className="h-4 w-4" />
    <span>CSV</span>
  </button>
</div>
```

---

### Task 4: Add Export for Product Table (Bonus)

**File:** `components/results/SalesView.tsx`

Also add export for the product performance table:

```typescript
const handleExportProductsCSV = () => {
  if (!data?.revenueByProduct) return

  const csvData = data.revenueByProduct.map(row => ({
    productId: row.productId,
    revenue: row.revenue,
    previousYear: row.previousYear || 0,
    variancePY: row.variancePY || 0,
    unitsSold: row.unitsSold,
    avgPrice: (row.revenue / row.unitsSold).toFixed(2)
  }))

  const columns = [
    { key: 'productId', header: 'Product' },
    { key: 'revenue', header: 'Revenue (AC)' },
    { key: 'previousYear', header: 'Revenue (PY)' },
    { key: 'variancePY', header: 'Variance (€)' },
    { key: 'unitsSold', header: 'Units Sold' },
    { key: 'avgPrice', header: 'Avg Price (€)' }
  ]

  const csv = arrayToCSV(csvData, columns)
  downloadCSV(csv, `products-summary-${scenarioId}.csv`)
}
```

Add button to product table header.

---

## UI Design

### Button Style
- Small, subtle button (not primary)
- Download icon + "CSV" text
- Gray color, darker on hover
- Positioned in section header, right-aligned

### Button States
- Default: Gray text/icon
- Hover: Darker gray, light background
- Disabled: If no data available (grayed out)

---

## Acceptance Criteria

- [ ] CSV utility functions created and working
- [ ] Finance View has CSV export button
- [ ] Sales View has CSV export button (monthly data)
- [ ] Sales View has CSV export for products table (bonus)
- [ ] Downloaded files have meaningful names (include scenarioId)
- [ ] CSV format is valid and opens correctly in Excel
- [ ] Numbers are not quoted (proper CSV format)
- [ ] Lint passes

---

## Testing Checklist

- [ ] Generate a scenario
- [ ] Click Finance CSV button → downloads `finance-summary-{id}.csv`
- [ ] Click Sales CSV button → downloads `sales-summary-{id}.csv`
- [ ] Open CSV in Excel → data displays correctly
- [ ] Open CSV in Google Sheets → data displays correctly
- [ ] Numbers can be summed (not treated as text)

---

## Notes

- Pure client-side implementation (no new API endpoints)
- Uses Blob API for download (works in all modern browsers)
- Filename includes scenarioId for traceability
- CSV uses proper quoting for strings with commas/quotes
