# Issue: Metric Naming & Scale Inconsistency

**Project:** ChainSights
**Priority:** Low (not launch-blocking)
**Type:** UX Consistency
**Date:** 2026-01-28

---

## Problem

Users see different names and scales for the same metrics in Quick Check vs. PDF Report, causing confusion.

### Current State

| Location | PDI Display | Scale |
|----------|-------------|-------|
| Quick Check | "Power Dynamics Index (PDI)" | 0-100 (e.g., 50) |
| PDF Report | "Decentralization Score" | 0-10 (e.g., 5/10) |

**User confusion:** "I got PDI 50 in Quick Check but Decentralization Score 2/10 in the Report – are these the same thing?"

---

## Affected Files

- `MetricTooltip.tsx` (line ~44) – tooltip definitions
- Quick Check result component – scale display
- PDF report template – already uses "Decentralization Score"

---

## Recommended Fix

### 1. Align Naming (MetricTooltip.tsx)

```typescript
// Before
PDI: {
  fullName: 'Power Dynamics Index',
  ...
}

// After
PDI: {
  fullName: 'Decentralization Score',
  abbreviation: 'PDI',
  description: 'How decentralized is voting power?'
}
```

### 2. Align All Metric Names

| Internal | User-Facing Name | Abbreviation |
|----------|------------------|--------------|
| HPR | Human Participation Rate | HPR |
| DEI | Delegate Engagement | DEI |
| PDI | **Decentralization Score** | PDI |
| GPI | Grassroots Participation | GPI |

### 3. Align Scale Display

Quick Check should display scores as X/10 (not 0-100) to match Report:

```
Before: HPR: 78
After:  Human Participation (HPR): 7.8/10
```

Or alternatively, show both:
```
Decentralization Score (PDI): 50/100 (5/10)
```

---

## Acceptance Criteria

- [ ] Quick Check tooltip for PDI says "Decentralization Score (PDI)"
- [ ] All four metrics have consistent user-facing names across Quick Check and Report
- [ ] Scale is visually consistent (either both X/10 or clear conversion shown)
- [ ] User can clearly see Quick Check metrics match Report metrics

---

## Notes

This is not launch-blocking but should be addressed in next iteration. Current state works, just causes minor confusion.
