# Results Page Polish - UX Specification

**Author:** Sally (UX Lead)
**Date:** December 2024
**Status:** Ready for Development

---

## Overview

Polish the results page based on PoC-Test-2 feedback from Mario and Matthias.

---

## Changes Required

### 1. Enhanced Header with Company Info

**Current:** Shows only scenario type badge, generation date, period, and API Docs link.

**New:** Add company name, sector, and size prominently.

```
┌─────────────────────────────────────────────────────────────────────────┐
│                                                                         │
│  🏢 Meridian Capital Holdings                                           │
│  Finance · Large Cap                                                    │
│                                                                         │
│  ┌──────────┐   Generated: Dec 11, 2024   36 months (Jan 2024 - Dec 2026) │
│  │ largecap │                                                           │
│  └──────────┘   ┌─────────────────────────────────────────────────┐     │
│                 │  📥 CSV (Sales) │ 📥 CSV (Finance) │ 📄 API Docs │     │
│                 └─────────────────────────────────────────────────┘     │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

**Implementation:**
- Fetch `companyName` from API (add to `/api/scenarios/[scenarioId]` response)
- Map `scenarioType` to sector/size using `TIER_CONFIGS` on frontend, OR return from API
- Display company name as prominent heading
- Show "Sector · Size" as subtitle

---

### 2. Remove/Hide Debug Information

**Remove completely:**
- "Tip: Save on Generation Costs" - Not relevant for end users

**Hide by default (collapsed accordion with "Technical Details" label):**
- "Multi-view consistency verified" badge - Still useful but not primary info
- Move from prominent position to collapsible section at bottom

**Keep but simplify:**
- "(generated in Xs)" - Keep in header but smaller/muted

---

### 3. Consolidate Export Links at Top

**Current:** CSV download buttons scattered inside each view component.

**New:** Add export action bar in header area, keep view-specific downloads too.

**Header Export Bar:**
```
┌─────────────────────────────────────────────────┐
│  📥 Sales CSV  │  📥 Finance CSV  │  📄 API Docs │
└─────────────────────────────────────────────────┘
```

- Sales CSV: Downloads sales summary (same as current button in SalesView)
- Finance CSV: Downloads finance summary (same as current button in FinanceView)
- API Docs: Link to /api-docs

**Note:** Keep existing per-view CSV buttons too (some users prefer contextual downloads).

---

## Updated Page Structure

```
┌─────────────────────────────────────────────────────────────────────────┐
│  [Header - Navigation]                                                  │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ┌───────────────────────────────────────────────────────────────────┐  │
│  │  🏢 Fresh Harvest Foods                                           │  │
│  │  Consumer Staples · Startup                                       │  │
│  │                                                                   │  │
│  │  ┌─────────┐  Generated: Dec 11, 2024  ·  36 months              │  │
│  │  │ startup │  (generated in 45s)                                  │  │
│  │  └─────────┘                                                      │  │
│  │                                                                   │  │
│  │  ┌──────────────────────────────────────────────────────────────┐│  │
│  │  │ 📥 Sales CSV  │  📥 Finance CSV  │  📄 API Docs              ││  │
│  │  └──────────────────────────────────────────────────────────────┘│  │
│  └───────────────────────────────────────────────────────────────────┘  │
│                                                                         │
│  ┌───────────────────────────────────────────────────────────────────┐  │
│  │  [Sales Tab] [Finance Tab]                                        │  │
│  │                                                                   │  │
│  │  [ View Content - Charts & Tables ]                               │  │
│  │                                                                   │  │
│  └───────────────────────────────────────────────────────────────────┘  │
│                                                                         │
│  ┌───────────────────────────────────────────────────────────────────┐  │
│  │  ▶ Technical Details (collapsed by default)                       │  │
│  │    └─ ✓ Multi-view consistency verified                          │  │
│  │       Revenue Reconciliation: Passed                              │  │
│  │       Payroll Alignment: Passed                                   │  │
│  │       COGS Matching: Passed                                       │  │
│  └───────────────────────────────────────────────────────────────────┘  │
│                                                                         │
│  [Footer]                                                               │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## API Changes Required

### GET /api/scenarios/[scenarioId]

**Add to response:**
```typescript
{
  scenarioType: string,      // existing
  generatedAt: string,       // existing
  period: string,            // existing
  generationDurationSeconds: number,  // existing
  // NEW:
  companyName: string,       // e.g., "Meridian Capital Holdings"
  sector: string,            // e.g., "Finance"
  size: string,              // e.g., "Large Cap"
}
```

**Implementation:** Look up `scenarioType` in `TIER_CONFIGS` to get sector/size, or store in DB.

---

## Files to Modify

1. `app/api/scenarios/[scenarioId]/route.ts` - Add companyName, sector, size to response
2. `app/results/[scenarioId]/page.tsx` - Redesign header, add export bar, move consistency badge
3. `components/ConsistencyBadge.tsx` - Make it collapsible "Technical Details" section

---

## Acceptance Criteria

- [ ] Company name displayed prominently as heading
- [ ] Sector and size shown as subtitle (e.g., "Finance · Large Cap")
- [ ] Export buttons (Sales CSV, Finance CSV, API Docs) consolidated in header
- [ ] "Tip: Save on Generation Costs" removed (if it exists)
- [ ] Consistency validation moved to collapsible "Technical Details" at bottom
- [ ] Generation time shown but muted/smaller
- [ ] Responsive on mobile
