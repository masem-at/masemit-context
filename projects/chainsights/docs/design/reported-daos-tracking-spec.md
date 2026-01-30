# Reported DAOs Tracking — Technical Spec

**Created:** 2024-12-19
**Author:** Winston (Architect)
**Status:** Ready for Implementation
**Assignee:** Amelia

---

## Overview

Track which DAOs have been analyzed, how many times, and their scores. Enables autocomplete in report generation dialog and provides analytics on popular DAOs.

---

## Database Schema

### New Table: `reported_daos`

```typescript
// src/lib/db/schema.ts

export const reportedDaos = pgTable('reported_daos', {
  spaceId: text('space_id').primaryKey(),
  name: text('name').notNull(),
  reportsGenerated: integer('reports_generated').default(1).notNull(),
  lastScore: integer('last_score'),           // 1-10, nullable until first report
  avgScore: real('avg_score'),                // Running average
  totalScoreSum: integer('total_score_sum'),  // For calculating avg without storing all scores
  lastReportedAt: timestamp('last_reported_at').defaultNow(),
  createdAt: timestamp('created_at').defaultNow(),
})
```

### Schema Notes

| Column | Type | Purpose |
|--------|------|---------|
| `space_id` | text PK | Snapshot space identifier |
| `name` | text | DAO display name from Snapshot |
| `reports_generated` | integer | Counter, starts at 1 |
| `last_score` | integer | Most recent decentralization score |
| `avg_score` | real | Running average of all scores |
| `total_score_sum` | integer | Sum of all scores (for avg calculation) |
| `last_reported_at` | timestamp | When last report was generated |
| `created_at` | timestamp | First report date |

### Average Score Calculation

```
avg_score = total_score_sum / reports_generated
```

On each new report:
```
total_score_sum += new_score
reports_generated += 1
avg_score = total_score_sum / reports_generated
```

---

## Backend Changes

### 1. Upsert Function

**File:** `src/lib/db/reported-daos.ts` (new file)

```typescript
import { db } from './index'
import { reportedDaos } from './schema'
import { sql, eq } from 'drizzle-orm'

export async function trackReportedDao(
  spaceId: string,
  name: string,
  score: number
): Promise<void> {
  await db
    .insert(reportedDaos)
    .values({
      spaceId,
      name,
      reportsGenerated: 1,
      lastScore: score,
      avgScore: score,
      totalScoreSum: score,
      lastReportedAt: new Date(),
    })
    .onConflictDoUpdate({
      target: reportedDaos.spaceId,
      set: {
        name,  // Update name in case it changed
        reportsGenerated: sql`${reportedDaos.reportsGenerated} + 1`,
        lastScore: score,
        totalScoreSum: sql`${reportedDaos.totalScoreSum} + ${score}`,
        avgScore: sql`(${reportedDaos.totalScoreSum} + ${score})::real / (${reportedDaos.reportsGenerated} + 1)`,
        lastReportedAt: new Date(),
      },
    })
}
```

### 2. Integration Point

**File:** `src/app/api/jobs/analyze/route.ts`

After successful analysis, before PDF generation:

```typescript
import { trackReportedDao } from '@/lib/db/reported-daos'

// After: const analysis = await analyzeGovernanceData(governanceData)
await trackReportedDao(
  report.spaceId,
  governanceData.space.name,
  analysis.decentralizationScore
)
```

### 3. API Endpoint for Autocomplete

**File:** `src/app/api/admin/reported-daos/route.ts` (new file)

```typescript
import { NextResponse } from 'next/server'
import { db } from '@/lib/db'
import { reportedDaos } from '@/lib/db/schema'
import { desc } from 'drizzle-orm'

export async function GET() {
  const daos = await db
    .select({
      spaceId: reportedDaos.spaceId,
      name: reportedDaos.name,
      reportsGenerated: reportedDaos.reportsGenerated,
      lastScore: reportedDaos.lastScore,
      avgScore: reportedDaos.avgScore,
      lastReportedAt: reportedDaos.lastReportedAt,
    })
    .from(reportedDaos)
    .orderBy(desc(reportedDaos.reportsGenerated))
    .limit(50)

  return NextResponse.json({ daos })
}
```

**Response:**

```json
{
  "daos": [
    {
      "spaceId": "arbitrumfoundation.eth",
      "name": "Arbitrum",
      "reportsGenerated": 5,
      "lastScore": 3,
      "avgScore": 2.8,
      "lastReportedAt": "2024-12-19T10:00:00Z"
    }
  ]
}
```

---

## Frontend Changes

### 1. Install Combobox Component

```bash
npx shadcn-ui@latest add command popover
```

### 2. Create Autocomplete Component

**File:** `src/components/dao-autocomplete.tsx` (new file)

```typescript
'use client'

import { useState, useEffect } from 'react'
import { Command, CommandEmpty, CommandGroup, CommandInput, CommandItem } from '@/components/ui/command'
import { Popover, PopoverContent, PopoverTrigger } from '@/components/ui/popover'
import { Button } from '@/components/ui/button'
import { Check, ChevronsUpDown } from 'lucide-react'
import { cn } from '@/lib/utils'

interface ReportedDao {
  spaceId: string
  name: string
  reportsGenerated: number
  lastScore: number | null
  avgScore: number | null
}

interface DaoAutocompleteProps {
  value: string
  onSelect: (spaceId: string, name: string) => void
}

export function DaoAutocomplete({ value, onSelect }: DaoAutocompleteProps) {
  const [open, setOpen] = useState(false)
  const [daos, setDaos] = useState<ReportedDao[]>([])
  const [inputValue, setInputValue] = useState(value)

  useEffect(() => {
    fetch('/api/admin/reported-daos')
      .then(res => res.json())
      .then(data => setDaos(data.daos))
  }, [])

  return (
    <Popover open={open} onOpenChange={setOpen}>
      <PopoverTrigger asChild>
        <Button
          variant="outline"
          role="combobox"
          aria-expanded={open}
          className="w-full justify-between"
        >
          {value || "Select or enter Space ID..."}
          <ChevronsUpDown className="ml-2 h-4 w-4 shrink-0 opacity-50" />
        </Button>
      </PopoverTrigger>
      <PopoverContent className="w-full p-0">
        <Command>
          <CommandInput
            placeholder="Search or enter new Space ID..."
            value={inputValue}
            onValueChange={(val) => {
              setInputValue(val)
              // Allow custom input
              onSelect(val, '')
            }}
          />
          <CommandEmpty>
            <div className="p-2 text-sm text-muted-foreground">
              New Space ID: "{inputValue}"
            </div>
          </CommandEmpty>
          <CommandGroup heading="Previously Reported">
            {daos.map((dao) => (
              <CommandItem
                key={dao.spaceId}
                onSelect={() => {
                  onSelect(dao.spaceId, dao.name)
                  setInputValue(dao.spaceId)
                  setOpen(false)
                }}
              >
                <Check
                  className={cn(
                    "mr-2 h-4 w-4",
                    value === dao.spaceId ? "opacity-100" : "opacity-0"
                  )}
                />
                <div className="flex flex-col">
                  <span className="font-medium">{dao.spaceId}</span>
                  <span className="text-sm text-muted-foreground">
                    {dao.name} • {dao.reportsGenerated} reports
                    {dao.lastScore && ` • Last: ${dao.lastScore}/10`}
                  </span>
                </div>
              </CommandItem>
            ))}
          </CommandGroup>
        </Command>
      </PopoverContent>
    </Popover>
  )
}
```

### 3. Integrate in Report Dialog

**File:** `src/app/admin/page.tsx` (modify existing)

Replace space ID input with autocomplete:

```typescript
import { DaoAutocomplete } from '@/components/dao-autocomplete'

// In the form:
<DaoAutocomplete
  value={spaceId}
  onSelect={(selectedSpaceId, selectedName) => {
    setSpaceId(selectedSpaceId)
    if (selectedName) {
      setDaoName(selectedName)  // Pre-fill name
    }
  }}
/>
```

---

## Migration

After schema update:

```bash
npm run db:push
```

---

## Acceptance Criteria

- [ ] `reported_daos` table created with all columns
- [ ] Upsert runs after successful report analysis
- [ ] `reports_generated` increments on repeat reports
- [ ] `last_score` and `avg_score` update correctly
- [ ] API returns DAOs sorted by report count
- [ ] Autocomplete shows previously reported DAOs
- [ ] Selecting from autocomplete pre-fills name
- [ ] Can still enter new Space IDs manually
- [ ] Works on admin page only (auth protected)

---

## Future Considerations (Not in Scope)

- Separate tracking for admin vs customer reports (add `source` column later)
- Historical score array instead of just avg
- Public "most analyzed DAOs" widget

---

## Files to Create/Modify

| File | Action |
|------|--------|
| `src/lib/db/schema.ts` | Add `reportedDaos` table |
| `src/lib/db/reported-daos.ts` | NEW — upsert function |
| `src/app/api/jobs/analyze/route.ts` | Add tracking call |
| `src/app/api/admin/reported-daos/route.ts` | NEW — GET endpoint |
| `src/components/dao-autocomplete.tsx` | NEW — autocomplete component |
| `src/app/admin/page.tsx` | Integrate autocomplete |

---

## Estimate

**3-4 hours total:**

| Task | Time |
|------|------|
| Schema + migration | 20 min |
| Upsert function | 30 min |
| Integration in analyze job | 15 min |
| API endpoint | 20 min |
| Autocomplete component | 1 hour |
| Integration in admin page | 30 min |
| Testing | 30 min |

---

*Spec complete. Ready for Amelia.*
