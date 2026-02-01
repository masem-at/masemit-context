# Story: Mini-API for Data Export

**Status:** Ready for Development
**Priority:** Medium (PoC Polish)
**Estimated Effort:** 1-2 hours

---

## Overview

Add clean REST API endpoints for programmatic data access, alongside a documentation page. This demonstrates professionalism - finance tools expect data integration capabilities.

**Key insight:** We already have `/api/views/finance` and `/api/views/sales` endpoints that return JSON. The Mini-API creates cleaner, more RESTful paths and adds proper documentation.

---

## User Story

As a developer or data analyst integrating with tellingCube,
I want to access scenario data via REST API,
So that I can pull data into my own tools (Python, Power BI, custom dashboards).

---

## Scope

### In Scope
- REST endpoints for Finance and Sales data (JSON)
- API Documentation page at `/api-docs`
- CORS headers for cross-origin requests
- Basic rate limiting awareness (note in docs)

### Out of Scope (Future)
- Authentication (API keys)
- Rate limiting implementation
- GraphQL endpoint
- Webhook notifications
- SDK packages

---

## API Design

### Finance Endpoint

```
GET /api/export/{scenarioId}/finance.json
```

**Response:** Same structure as existing `FinanceViewResponse`

```json
{
  "plSummary": {
    "revenue": { "actual": 1234567, "previousYear": 1100000, ... },
    "cogs": { ... },
    "payroll": { ... },
    "profit": { ... }
  },
  "monthlyCashFlow": [
    { "month": "2024-01-01", "cashIn": 123456, "cashOut": 98765, "netCash": 24691, ... },
    ...
  ],
  "costBreakdown": {
    "payroll": 345678,
    "payrollPercent": 28.5,
    ...
  },
  "marginAnalysis": {
    "grossMargin": 42.3,
    "netMargin": 12.5,
    ...
  }
}
```

### Sales Endpoint

```
GET /api/export/{scenarioId}/sales.json
```

**Response:** Same structure as existing `SalesViewResponse`

```json
{
  "revenueByMonth": [
    { "month": "2024-01-01", "revenue": 98765, "previousYear": 87654, "plan": 100000, "forecast": 95000, ... },
    ...
  ],
  "revenueByProduct": [
    { "productId": "PROD-001", "revenue": 234567, "unitsSold": 1234, ... },
    ...
  ],
  "topCustomers": [
    { "counterpartyId": "CUST-001", "totalRevenue": 45678, "transactionCount": 23 },
    ...
  ],
  "trendAnalysis": {
    "avgGrowthRate": 5.2,
    "maxMonth": "2024-08",
    "minMonth": "2024-02"
  }
}
```

### Error Responses

```json
{
  "error": {
    "message": "Scenario not found",
    "code": "SCENARIO_NOT_FOUND"
  }
}
```

Error codes:
- `INVALID_SCENARIO_ID` (400) - Invalid UUID format
- `SCENARIO_NOT_FOUND` (404) - Scenario doesn't exist
- `QUERY_ERROR` (500) - Database error

---

## Tasks

### Task 1: Create Finance Export Endpoint

**Create file:** `app/api/export/[scenarioId]/finance.json/route.ts`

```typescript
import { NextRequest, NextResponse } from 'next/server'
import {
  getPLSummary,
  getMonthlyCashFlow,
  getCostBreakdown,
  getMarginAnalysis,
} from '@/lib/queries/finance-queries'
import type { FinanceViewResponse } from '@/types/api'

/**
 * GET /api/export/{scenarioId}/finance.json
 *
 * Public API endpoint for Finance view data export.
 * Returns financial metrics in JSON format (visualizations inspired by IBCS©).
 */
export async function GET(
  request: NextRequest,
  { params }: { params: Promise<{ scenarioId: string }> }
): Promise<NextResponse> {
  try {
    const { scenarioId } = await params

    // Validate UUID format
    const uuidRegex = /^[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}$/i
    if (!uuidRegex.test(scenarioId)) {
      return NextResponse.json(
        { error: { message: 'Invalid scenarioId format', code: 'INVALID_SCENARIO_ID' } },
        { status: 400 }
      )
    }

    // Execute queries
    const [plSummary, monthlyCashFlow, costBreakdown, marginAnalysis] = await Promise.all([
      getPLSummary(scenarioId),
      getMonthlyCashFlow(scenarioId),
      getCostBreakdown(scenarioId),
      getMarginAnalysis(scenarioId),
    ])

    // Check if scenario exists (plSummary would be empty/null)
    if (!plSummary || plSummary.revenue.actual === 0) {
      return NextResponse.json(
        { error: { message: 'Scenario not found or has no data', code: 'SCENARIO_NOT_FOUND' } },
        { status: 404 }
      )
    }

    const response: FinanceViewResponse = {
      plSummary,
      monthlyCashFlow,
      costBreakdown,
      marginAnalysis,
    }

    // Return with CORS headers
    return NextResponse.json(response, {
      status: 200,
      headers: {
        'Access-Control-Allow-Origin': '*',
        'Access-Control-Allow-Methods': 'GET, OPTIONS',
        'Content-Type': 'application/json',
      },
    })
  } catch (error) {
    console.error('Finance export failed:', error)
    return NextResponse.json(
      { error: { message: 'Failed to fetch finance data', code: 'QUERY_ERROR' } },
      { status: 500 }
    )
  }
}

// Handle CORS preflight
export async function OPTIONS() {
  return NextResponse.json({}, {
    headers: {
      'Access-Control-Allow-Origin': '*',
      'Access-Control-Allow-Methods': 'GET, OPTIONS',
      'Access-Control-Allow-Headers': 'Content-Type',
    },
  })
}
```

---

### Task 2: Create Sales Export Endpoint

**Create file:** `app/api/export/[scenarioId]/sales.json/route.ts`

```typescript
import { NextRequest, NextResponse } from 'next/server'
import {
  getRevenueByMonth,
  getRevenueByProduct,
  getTopCustomers,
  getSalesTrendAnalysis,
} from '@/lib/queries/sales-queries'
import type { SalesViewResponse } from '@/types/api'

/**
 * GET /api/export/{scenarioId}/sales.json
 *
 * Public API endpoint for Sales view data export.
 * Returns sales metrics in JSON format (visualizations inspired by IBCS©).
 */
export async function GET(
  request: NextRequest,
  { params }: { params: Promise<{ scenarioId: string }> }
): Promise<NextResponse> {
  try {
    const { scenarioId } = await params

    // Validate UUID format
    const uuidRegex = /^[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}$/i
    if (!uuidRegex.test(scenarioId)) {
      return NextResponse.json(
        { error: { message: 'Invalid scenarioId format', code: 'INVALID_SCENARIO_ID' } },
        { status: 400 }
      )
    }

    // Parse optional topN parameter (default: 10)
    const topN = parseInt(request.nextUrl.searchParams.get('topN') || '10', 10)

    // Execute queries
    const [revenueByMonth, revenueByProduct, topCustomers, trendAnalysis] = await Promise.all([
      getRevenueByMonth(scenarioId),
      getRevenueByProduct(scenarioId),
      getTopCustomers(scenarioId, topN),
      getSalesTrendAnalysis(scenarioId),
    ])

    // Check if scenario exists
    if (!revenueByMonth || revenueByMonth.length === 0) {
      return NextResponse.json(
        { error: { message: 'Scenario not found or has no data', code: 'SCENARIO_NOT_FOUND' } },
        { status: 404 }
      )
    }

    const response: SalesViewResponse = {
      revenueByMonth,
      revenueByProduct,
      topCustomers,
      trendAnalysis,
    }

    // Return with CORS headers
    return NextResponse.json(response, {
      status: 200,
      headers: {
        'Access-Control-Allow-Origin': '*',
        'Access-Control-Allow-Methods': 'GET, OPTIONS',
        'Content-Type': 'application/json',
      },
    })
  } catch (error) {
    console.error('Sales export failed:', error)
    return NextResponse.json(
      { error: { message: 'Failed to fetch sales data', code: 'QUERY_ERROR' } },
      { status: 500 }
    )
  }
}

// Handle CORS preflight
export async function OPTIONS() {
  return NextResponse.json({}, {
    headers: {
      'Access-Control-Allow-Origin': '*',
      'Access-Control-Allow-Methods': 'GET, OPTIONS',
      'Access-Control-Allow-Headers': 'Content-Type',
    },
  })
}
```

---

### Task 3: Create API Documentation Page

**Create file:** `app/api-docs/page.tsx`

```tsx
import Link from 'next/link'
import { Header } from '@/components/layout/Header'
import { FileJson, Download, ExternalLink, Code } from 'lucide-react'

export const metadata = {
  title: 'API Documentation | tellingCube',
  description: 'REST API documentation for programmatic access to tellingCube scenario data',
}

export default function APIDocsPage() {
  return (
    <>
      <Header />
      <main className="min-h-screen bg-gray-50">
        <div className="max-w-4xl mx-auto px-6 py-12">
          {/* Header */}
          <div className="mb-12">
            <h1 className="text-3xl font-bold text-gray-900 mb-4">API Documentation</h1>
            <p className="text-lg text-gray-600">
              Access your scenario data programmatically via our REST API.
              Perfect for integration with Power BI, Python scripts, or custom dashboards.
            </p>
          </div>

          {/* Base URL */}
          <section className="bg-white rounded-lg border border-gray-200 p-6 mb-8">
            <h2 className="text-lg font-semibold text-gray-900 mb-3">Base URL</h2>
            <code className="block bg-gray-100 px-4 py-3 rounded-md font-mono text-sm">
              https://tellingcube.com/api/export
            </code>
            <p className="text-sm text-gray-500 mt-2">
              All endpoints return JSON. CORS is enabled for cross-origin requests.
            </p>
          </section>

          {/* Authentication */}
          <section className="bg-blue-50 border border-blue-200 rounded-lg p-6 mb-8">
            <h2 className="text-lg font-semibold text-blue-900 mb-2">Authentication</h2>
            <p className="text-blue-800">
              <strong>Current (PoC):</strong> No authentication required. Access is open.
            </p>
            <p className="text-sm text-blue-700 mt-2">
              Future versions will support API keys for secure access.
            </p>
          </section>

          {/* Endpoints */}
          <section className="space-y-6">
            <h2 className="text-xl font-semibold text-gray-900">Endpoints</h2>

            {/* Finance Endpoint */}
            <div className="bg-white rounded-lg border border-gray-200 overflow-hidden">
              <div className="bg-gray-50 px-6 py-4 border-b border-gray-200">
                <div className="flex items-center gap-3">
                  <span className="px-2 py-1 bg-green-100 text-green-700 text-xs font-bold rounded">
                    GET
                  </span>
                  <code className="font-mono text-sm">/api/export/{'{scenarioId}'}/finance.json</code>
                </div>
              </div>
              <div className="p-6">
                <p className="text-gray-600 mb-4">
                  Returns aggregated finance data including P&L summary, monthly cash flow,
                  cost breakdown, and margin analysis. Visualizations are inspired by IBCS©
                  with AC, PY, PL, and FC comparisons.
                </p>

                <h4 className="text-sm font-semibold text-gray-700 mb-2">Example Request</h4>
                <pre className="bg-gray-100 px-4 py-3 rounded-md font-mono text-xs overflow-x-auto mb-4">
{`curl https://tellingcube.com/api/export/abc12345-1234-5678-9abc-def012345678/finance.json`}
                </pre>

                <h4 className="text-sm font-semibold text-gray-700 mb-2">Response Structure</h4>
                <pre className="bg-gray-100 px-4 py-3 rounded-md font-mono text-xs overflow-x-auto">
{`{
  "plSummary": {
    "revenue": { "actual": 1234567, "previousYear": 1100000, "plan": 1300000, ... },
    "cogs": { "actual": 456789, ... },
    "payroll": { "actual": 234567, ... },
    "profit": { "actual": 543211, ... }
  },
  "monthlyCashFlow": [
    { "month": "2024-01-01", "cashIn": 123456, "cashOut": 98765, "netCash": 24691 },
    ...
  ],
  "costBreakdown": { "payroll": 234567, "payrollPercent": 19.0, ... },
  "marginAnalysis": { "grossMargin": 63.0, "netMargin": 44.0, ... }
}`}
                </pre>
              </div>
            </div>

            {/* Sales Endpoint */}
            <div className="bg-white rounded-lg border border-gray-200 overflow-hidden">
              <div className="bg-gray-50 px-6 py-4 border-b border-gray-200">
                <div className="flex items-center gap-3">
                  <span className="px-2 py-1 bg-green-100 text-green-700 text-xs font-bold rounded">
                    GET
                  </span>
                  <code className="font-mono text-sm">/api/export/{'{scenarioId}'}/sales.json</code>
                </div>
              </div>
              <div className="p-6">
                <p className="text-gray-600 mb-4">
                  Returns aggregated sales data including monthly revenue, product performance,
                  top customers, and trend analysis. Includes variance data inspired by IBCS©.
                </p>

                <h4 className="text-sm font-semibold text-gray-700 mb-2">Parameters</h4>
                <table className="w-full text-sm mb-4">
                  <thead>
                    <tr className="border-b">
                      <th className="text-left py-2 font-semibold">Name</th>
                      <th className="text-left py-2 font-semibold">Type</th>
                      <th className="text-left py-2 font-semibold">Description</th>
                    </tr>
                  </thead>
                  <tbody>
                    <tr className="border-b">
                      <td className="py-2"><code>topN</code></td>
                      <td className="py-2">integer</td>
                      <td className="py-2">Number of top customers to return (default: 10)</td>
                    </tr>
                  </tbody>
                </table>

                <h4 className="text-sm font-semibold text-gray-700 mb-2">Example Request</h4>
                <pre className="bg-gray-100 px-4 py-3 rounded-md font-mono text-xs overflow-x-auto mb-4">
{`curl "https://tellingcube.com/api/export/abc12345-1234-5678-9abc-def012345678/sales.json?topN=5"`}
                </pre>

                <h4 className="text-sm font-semibold text-gray-700 mb-2">Response Structure</h4>
                <pre className="bg-gray-100 px-4 py-3 rounded-md font-mono text-xs overflow-x-auto">
{`{
  "revenueByMonth": [
    { "month": "2024-01-01", "revenue": 98765, "previousYear": 87654, "plan": 100000, ... },
    ...
  ],
  "revenueByProduct": [
    { "productId": "PROD-001", "revenue": 234567, "unitsSold": 1234, ... },
    ...
  ],
  "topCustomers": [
    { "counterpartyId": "CUST-001", "totalRevenue": 45678, "transactionCount": 23 },
    ...
  ],
  "trendAnalysis": { "avgGrowthRate": 5.2, "maxMonth": "2024-08", "minMonth": "2024-02" }
}`}
                </pre>
              </div>
            </div>
          </section>

          {/* Error Codes */}
          <section className="mt-8">
            <h2 className="text-xl font-semibold text-gray-900 mb-4">Error Codes</h2>
            <div className="bg-white rounded-lg border border-gray-200 overflow-hidden">
              <table className="w-full text-sm">
                <thead className="bg-gray-50">
                  <tr className="border-b">
                    <th className="text-left px-6 py-3 font-semibold">Code</th>
                    <th className="text-left px-6 py-3 font-semibold">HTTP Status</th>
                    <th className="text-left px-6 py-3 font-semibold">Description</th>
                  </tr>
                </thead>
                <tbody className="divide-y divide-gray-200">
                  <tr>
                    <td className="px-6 py-3 font-mono">INVALID_SCENARIO_ID</td>
                    <td className="px-6 py-3">400</td>
                    <td className="px-6 py-3">scenarioId is not a valid UUID</td>
                  </tr>
                  <tr>
                    <td className="px-6 py-3 font-mono">SCENARIO_NOT_FOUND</td>
                    <td className="px-6 py-3">404</td>
                    <td className="px-6 py-3">Scenario doesn't exist or has no data</td>
                  </tr>
                  <tr>
                    <td className="px-6 py-3 font-mono">QUERY_ERROR</td>
                    <td className="px-6 py-3">500</td>
                    <td className="px-6 py-3">Internal server error</td>
                  </tr>
                </tbody>
              </table>
            </div>
          </section>

          {/* Usage Examples */}
          <section className="mt-8">
            <h2 className="text-xl font-semibold text-gray-900 mb-4">Usage Examples</h2>

            <div className="space-y-4">
              {/* Python Example */}
              <div className="bg-white rounded-lg border border-gray-200 overflow-hidden">
                <div className="bg-gray-50 px-6 py-3 border-b border-gray-200 flex items-center gap-2">
                  <Code className="h-4 w-4 text-gray-500" />
                  <span className="font-semibold text-sm">Python</span>
                </div>
                <pre className="p-6 font-mono text-xs overflow-x-auto">
{`import requests

SCENARIO_ID = "abc12345-1234-5678-9abc-def012345678"
BASE_URL = "https://tellingcube.com/api/export"

# Get finance data
finance = requests.get(f"{BASE_URL}/{SCENARIO_ID}/finance.json").json()
print(f"Total Revenue: €{finance['plSummary']['revenue']['actual']:,}")

# Get sales data
sales = requests.get(f"{BASE_URL}/{SCENARIO_ID}/sales.json").json()
for product in sales['revenueByProduct'][:5]:
    print(f"{product['productId']}: €{product['revenue']:,}")`}
                </pre>
              </div>

              {/* JavaScript Example */}
              <div className="bg-white rounded-lg border border-gray-200 overflow-hidden">
                <div className="bg-gray-50 px-6 py-3 border-b border-gray-200 flex items-center gap-2">
                  <Code className="h-4 w-4 text-gray-500" />
                  <span className="font-semibold text-sm">JavaScript</span>
                </div>
                <pre className="p-6 font-mono text-xs overflow-x-auto">
{`const SCENARIO_ID = "abc12345-1234-5678-9abc-def012345678";
const BASE_URL = "https://tellingcube.com/api/export";

// Get finance data
const finance = await fetch(\`\${BASE_URL}/\${SCENARIO_ID}/finance.json\`)
  .then(res => res.json());

console.log(\`Total Revenue: €\${finance.plSummary.revenue.actual.toLocaleString()}\`);

// Get sales data
const sales = await fetch(\`\${BASE_URL}/\${SCENARIO_ID}/sales.json\`)
  .then(res => res.json());

sales.revenueByProduct.slice(0, 5).forEach(p => {
  console.log(\`\${p.productId}: €\${p.revenue.toLocaleString()}\`);
});`}
                </pre>
              </div>

              {/* Power BI Note */}
              <div className="bg-yellow-50 border border-yellow-200 rounded-lg p-6">
                <h3 className="font-semibold text-yellow-900 mb-2">Power BI Integration</h3>
                <p className="text-yellow-800 text-sm">
                  Use <strong>Get Data → Web</strong> and enter the API URL.
                  Power BI will automatically parse the JSON response.
                  For refreshable reports, use Web.Contents with the API URL.
                </p>
              </div>
            </div>
          </section>

          {/* IBCS Note */}
          <section className="mt-8 bg-gray-100 rounded-lg p-6">
            <h2 className="text-lg font-semibold text-gray-900 mb-3">
              About Scenario Notation (inspired by IBCS©)
            </h2>
            <p className="text-gray-700 mb-3">
              Our data uses scenario notation inspired by IBCS© (International Business Communication Standards):
            </p>
            <ul className="space-y-2 text-sm text-gray-600">
              <li><strong>AC (Actual)</strong> - Current year actual figures (2024)</li>
              <li><strong>PY (Previous Year)</strong> - Last year's actual figures (2023)</li>
              <li><strong>PL (Plan)</strong> - Ambitious budget targets set before the period</li>
              <li><strong>FC (Forecast)</strong> - Realistic expectations updated during the period</li>
            </ul>
          </section>

          {/* Back Link */}
          <div className="mt-12 text-center">
            <Link
              href="/scenarios"
              className="text-blue-600 hover:text-blue-700 hover:underline"
            >
              ← Back to My Scenarios
            </Link>
          </div>
        </div>
      </main>
    </>
  )
}
```

---

### Task 4: Add Link to API Docs (Header or Footer)

**Option A: Add to Results page header** (recommended for PoC)

In `app/results/[scenarioId]/page.tsx`, add a small "API" link near the CSV buttons.

**Option B: Add to global navigation** (future enhancement)

Could add an "API Docs" link to the Header component.

For PoC, just make sure the `/api-docs` page is accessible via direct URL.

---

## Acceptance Criteria

- [ ] Finance export endpoint works: `GET /api/export/{id}/finance.json`
- [ ] Sales export endpoint works: `GET /api/export/{id}/sales.json`
- [ ] Both endpoints return proper CORS headers
- [ ] Invalid scenarioId returns 400 error
- [ ] Non-existent scenario returns 404 error
- [ ] API Documentation page at `/api-docs` is complete
- [ ] Documentation includes example requests and responses
- [ ] Documentation includes Python and JavaScript examples
- [ ] Lint passes

---

## Testing Checklist

- [ ] Generate a scenario
- [ ] Call finance endpoint with valid scenarioId → returns JSON
- [ ] Call sales endpoint with valid scenarioId → returns JSON
- [ ] Call with invalid UUID → returns 400
- [ ] Call with non-existent UUID → returns 404
- [ ] Test from different origin (CORS) → works
- [ ] Visit `/api-docs` → documentation displays
- [ ] Copy example code → works in browser console

---

## Notes

- Reuses existing query functions from `lib/queries/`
- Same data structure as internal API - just cleaner paths
- CORS enabled for cross-origin requests (Power BI, Postman, etc.)
- No authentication for PoC - can add API keys later
- Documentation page follows same visual style as rest of app
