# DEV TICKET: Methodology Version Marker in Historical Charts

**Priority:** High — must ship before next public communication
**Affected:** All GVS trend charts (Matrix Details page, PDF reports)
**Reporter:** Mario
**Date:** February 12, 2026 (reactivated — originally Feb 11)

---

## Context

The combined rollout of three fixes on Feb 12, 2026 caused significant ranking changes across all DAOs:

1. **Snapshot Client Cache Fix** — 4→1 API calls, HPR now calculates correctly for 26/47 DAOs
2. **DEI Fix** — Top 20% Voter Engagement proxy replaces broken vp_by_strategy delegate detection. DEI now differentiated (2.9-10.0) instead of constant 5.0
3. **Checkbox Voting Detection Phase 1** — GPI now penalises rapid/undiversified voting

**Impact is massive — examples:**

| DAO | GVS Before | GVS After | Change | Rank Before | Rank After |
|-----|-----------|----------|--------|------------|-----------|
| StakeDAO | 7.1 | 8.3 | +1.2 | #9 | #1 |
| Radiant Capital | 6.8 | 8.0 | +1.2 | #15 | #2 |
| Alchemix | 6.7 | 7.9 | +1.2 | ~#16 | #3 |
| Aave DAO | 6.3 | 5.8 | -0.5 | #23 | #22 |
| GnosisDAO | 6.3 | 5.8 | -0.5 | #24 | #23 |
| Lido | 6.6 | 6.8 | +0.2 | #19 | #26 |

Without visual explanation, users will see dramatic ranking shifts and assume data errors.

---

## Requirement

Add a **vertical marker line** at the methodology change date in all historical trend charts, with a tooltip explaining the change.

### Visual Spec

- **Marker type:** Vertical dashed line spanning full chart height
- **Color:** Neutral grey (#6b7280) — not red/green, it's informational
- **Date:** February 12, 2026
- **Label:** Small "v1.1" text or ℹ️ icon at top of line

### Tooltip Content (on hover/tap)

```
Methodology Update (v1.1)
February 12, 2026

Three scoring improvements applied:
• DEI now measures top voter engagement
  (previously constant at 5.0)
• Improved data pipeline reliability
• Vote quality signals added to GPI

Score changes on this date reflect improved
methodology, not governance changes.
```

### Where It Appears

| Location | Implementation |
|----------|---------------|
| Matrix Details — GVS trend chart | Recharts ReferenceLine |
| Matrix Details — HPR/DEI/PDI/GPI sub-charts | Same ReferenceLine on each |
| PDF Report — Historical Trends Analysis | Static marker line + footnote |

### Recharts Implementation

```jsx
import { ReferenceLine } from 'recharts';

<ReferenceLine
  x="2026-02-12"
  stroke="#6b7280"
  strokeDasharray="4 4"
  label={{
    value: "v1.1",
    position: "top",
    fontSize: 11,
    fill: "#6b7280"
  }}
/>
```

### PDF Footnote

Below the Historical Trends chart:

> *Dashed line indicates methodology update (v1.1, Feb 12 2026): Improved delegate engagement measurement, data pipeline reliability, and vote quality signals. Score changes at this date reflect improved methodology, not governance changes.*

---

## Optional: methodology_versions Table

For future-proofing, store methodology versions in the database:

```sql
CREATE TABLE methodology_versions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  version VARCHAR(20) NOT NULL,
  effective_date DATE NOT NULL,
  short_label VARCHAR(50) NOT NULL,
  tooltip_text TEXT NOT NULL,
  created_at TIMESTAMP NOT NULL DEFAULT NOW()
);

INSERT INTO methodology_versions (version, effective_date, short_label, tooltip_text) VALUES
('v1.1', '2026-02-12',
 'Scoring improvements',
 'Three scoring improvements: DEI now measures top voter engagement (previously constant), improved data pipeline reliability, and vote quality signals added to GPI.');
```

Charts query this table and render markers dynamically — no code change needed for future updates.

---

## Effort Estimate

| Component | Hours |
|-----------|-------|
| Recharts ReferenceLine + tooltip (all charts) | 2h |
| PDF chart marker + footnote | 1h |
| methodology_versions table (recommended) | 1h |
| Testing | 1h |
| **Total** | **~5h** |

---

## Acceptance Criteria

- [ ] Vertical dashed marker visible at Feb 12 on all GVS trend charts
- [ ] Hover/tap tooltip explains the three methodology changes
- [ ] PDF report includes marker line and footnote
- [ ] Marker does NOT appear on charts where date range excludes Feb 12
- [ ] Mobile: tooltip accessible via tap
- [ ] Ships before Jenya_K receives the Lido Governance Audit report
