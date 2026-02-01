# Tech Spec: ZIP Bundle Downloads

## Overview

Add 3 new download options to the "More Exports" dropdown on the results page that bundle existing exports into ZIP files.

## New Options

| Option | Contents | Filename |
|--------|----------|----------|
| All JSON (.zip) | finance.json, sales.json, events.json, hr.json | `{company}-json.zip` |
| All CSV (.zip) | events.csv, hr.csv | `{company}-csv.zip` |
| All Data (.zip) | All 6 files above | `{company}-all.zip` |

## Approach: Client-Side with JSZip

Fetch existing API endpoints in parallel, bundle with JSZip, trigger browser download. No new API routes needed.

### Install

```bash
npm install jszip
```

### Implementation

**File:** `lib/utils/zip-export.ts` (new)

```typescript
import JSZip from 'jszip'

type BundleType = 'json' | 'csv' | 'all'

interface ExportFile {
  filename: string
  url: string
}

function getFiles(scenarioId: string, type: BundleType): ExportFile[] {
  const base = `/api/export/${scenarioId}`
  const jsonFiles: ExportFile[] = [
    { filename: 'finance.json', url: `${base}/finance.json` },
    { filename: 'sales.json', url: `${base}/sales.json` },
    { filename: 'events.json', url: `${base}/events.json` },
    { filename: 'hr.json', url: `${base}/hr.json` },
  ]
  const csvFiles: ExportFile[] = [
    { filename: 'events.csv', url: `${base}/events.csv` },
    { filename: 'hr.csv', url: `${base}/hr.csv` },
  ]

  if (type === 'json') return jsonFiles
  if (type === 'csv') return csvFiles
  return [...jsonFiles, ...csvFiles]
}

export async function downloadZipBundle(
  scenarioId: string,
  companyName: string,
  type: BundleType
): Promise<void> {
  const files = getFiles(scenarioId, type)
  const responses = await Promise.all(files.map(f => fetch(f.url)))
  const zip = new JSZip()

  for (let i = 0; i < files.length; i++) {
    const blob = await responses[i].blob()
    zip.file(files[i].filename, blob)
  }

  const suffix = type === 'json' ? 'json' : type === 'csv' ? 'csv' : 'all'
  const zipBlob = await zip.generateAsync({ type: 'blob' })
  const url = URL.createObjectURL(zipBlob)
  const a = document.createElement('a')
  a.href = url
  a.download = `${companyName}-${suffix}.zip`
  a.click()
  URL.revokeObjectURL(url)
}
```

### UI Changes

**File:** `app/results/[scenarioId]/page.tsx`

Add to the "More Exports" dropdown after existing items, separated by a divider:

```
── Bundle Downloads ──────
📦 All JSON (.zip)
📦 All CSV (.zip)
📦 All Data (.zip)
```

Each option calls `downloadZipBundle(scenarioId, companyName, type)`.

Use `'use client'` -- the page likely already is client-rendered. Import dynamically if needed.

### Dark Mode

Follow existing dropdown dark mode classes already present in the component.

## Status: Implementation Complete

## Acceptance Criteria

- [x] AC1: "All JSON (.zip)" downloads a ZIP containing finance.json, sales.json, events.json, hr.json
- [x] AC2: "All CSV (.zip)" downloads a ZIP containing events.csv, hr.csv
- [x] AC3: "All Data (.zip)" downloads a ZIP containing all 6 files
- [x] AC4: ZIP filenames include company name
- [x] AC5: Visual divider separates bundle options from individual exports
- [x] AC6: Loading state shown during ZIP generation (button text or spinner)
- [x] AC7: Error handling if any fetch fails
- [x] AC8: Works in dark mode
