# Healthcare Sector Research

**Date:** January 24, 2026
**Author:** Mary (Business Analyst)
**Status:** Ready for Architecture Review
**Company Profile:** MedTech Diagnostics AG (MidCap)

---

## Executive Summary

This document defines the Healthcare sector configuration for tellingCube, enabling realistic medical/healthcare business data generation. The profile is designed for a MidCap medical diagnostics company with clinical-specific HR roles.

---

## 1. Sector Configuration

### Basic Definition

```typescript
'healthcare': {
  sector: 'healthcare',
  sectorCode: 'healthcare',
  label: 'Healthcare',
  description: 'Medical devices, diagnostics, healthcare services',
  icon: 'Heart', // Lucide icon - alternatives: Stethoscope, Activity
}
```

### Company Profile: MedTech Diagnostics AG

| Attribute | Value |
|-----------|-------|
| **Name** | MedTech Diagnostics AG |
| **Size** | MidCap |
| **Type** | Medical diagnostics & laboratory services |
| **Region** | Europe (DACH) |
| **Employees** | 65-95 (target: ~78) |

---

## 2. Financial Constraints

Based on healthcare industry benchmarks (NYU Stern, McKinsey Health):

| Metric | Range | Notes |
|--------|-------|-------|
| **Cost/Income Ratio** | 0.82 - 0.92 | Higher labor costs than manufacturing |
| **Payroll Share** | 0.45 - 0.55 | Labor-intensive (clinical staff) |
| **COGS Share** | 0.25 - 0.40 | Medical supplies, reagents, equipment |
| **Other Share** | 0.12 - 0.22 | Compliance, insurance, facility costs |
| **Gross Margin** | 0.55 - 0.70 | Higher than industrials due to specialized services |
| **Net Margin** | 0.08 - 0.18 | Moderate, varies by service mix |

### Comparison to Existing Sectors

| Sector | Gross Margin | Net Margin | Payroll Share |
|--------|-------------|------------|---------------|
| Consumer Staples | 35-50% | -60% to +12% | 15-25% |
| Industrials | 25-38% | 4-12% | 28-38% |
| Financials | 60-70% | 12-25% | 38-48% |
| **Healthcare** | **55-70%** | **8-18%** | **45-55%** |

**Key Insight:** Healthcare has high payroll (like Financials) but also significant COGS (medical supplies).

---

## 3. Department Structure

### Clinical Departments

| Department | Code | Typical Headcount % |
|------------|------|---------------------|
| Laboratory Services | LAB | 25-30% |
| Radiology/Imaging | RAD | 15-20% |
| Clinical Operations | CLIN | 20-25% |
| Quality Assurance | QA | 8-12% |

### Administrative Departments

| Department | Code | Typical Headcount % |
|------------|------|---------------------|
| Administration | ADMIN | 10-15% |
| IT & Digital Health | IT | 5-8% |
| Finance & Billing | FIN | 5-8% |
| Executive | EXEC | 3-5% |

---

## 4. Clinical-Specific HR Roles

### Role Types by Level

| Level | Generic Role | Healthcare-Specific Roles |
|-------|--------------|---------------------------|
| **Executive** | C-Suite | Chief Medical Officer (CMO), Chief Operating Officer, Medical Director |
| **Manager** | Department Head | Lab Director, Radiology Manager, Clinical Operations Manager, QA Manager |
| **Specialist** | Senior Individual Contributor | Senior Technologist, Clinical Specialist, Quality Engineer, Senior Analyst |
| **Junior** | Entry Level | Lab Technician, Radiology Tech, Medical Assistant, Administrative Coordinator |

### Detailed Role Definitions

```typescript
// Healthcare-specific roles for HR generation
const HEALTHCARE_ROLES = {
  executive: [
    'Chief Medical Officer',
    'Chief Operating Officer',
    'Medical Director',
    'VP Clinical Operations',
  ],
  manager: [
    'Laboratory Director',
    'Radiology Manager',
    'Clinical Operations Manager',
    'Quality Assurance Manager',
    'IT Manager',
    'Finance Manager',
  ],
  specialist: [
    'Senior Lab Technologist',
    'Clinical Specialist',
    'Quality Engineer',
    'Biomedical Engineer',
    'Senior Radiologic Technologist',
    'Compliance Specialist',
    'Data Analyst',
  ],
  junior: [
    'Laboratory Technician',
    'Radiology Technician',
    'Medical Assistant',
    'Phlebotomist',
    'Administrative Coordinator',
    'Billing Specialist',
    'IT Support Technician',
  ],
}
```

### Salary Ranges (EUR monthly, MidCap)

| Level | Min | Max | Notes |
|-------|-----|-----|-------|
| Executive | 12,000 | 22,000 | CMO/Medical Director premium |
| Manager | 6,000 | 11,000 | Clinical managers slightly higher |
| Specialist | 4,000 | 7,000 | Certified technologists |
| Junior | 2,800 | 4,200 | Entry-level clinical staff |

---

## 5. Revenue Events

### Primary Revenue Streams

| Event Type | Description | Typical % of Revenue |
|------------|-------------|---------------------|
| **Diagnostic Tests** | Laboratory analysis, blood work, pathology | 45-55% |
| **Imaging Services** | X-ray, CT, MRI, ultrasound | 25-35% |
| **Consulting Services** | Specialist consultations, second opinions | 10-15% |
| **Equipment Sales/Leasing** | Diagnostic equipment to clinics | 5-10% |

### Customer Types

- Hospitals and health systems
- Private medical practices
- Insurance companies (direct billing)
- Research institutions
- Corporate wellness programs

---

## 6. Cost Events

### Cost of Goods Sold (COGS)

| Cost Category | % of COGS | Description |
|---------------|-----------|-------------|
| Medical Supplies | 40-50% | Reagents, test kits, consumables |
| Equipment Depreciation | 20-30% | Diagnostic equipment amortization |
| External Lab Services | 15-20% | Outsourced specialized tests |
| Quality Control Materials | 5-10% | Calibration, standards |

### Operating Expenses

| Expense Category | % of OpEx | Description |
|------------------|-----------|-------------|
| Payroll | 60-70% | Clinical and admin staff |
| Facility Costs | 10-15% | Rent, utilities, maintenance |
| Insurance & Compliance | 8-12% | Liability, certifications |
| IT & Software | 5-8% | LIS, PACS systems |
| Marketing | 3-5% | B2B marketing to clinics |

---

## 7. Seasonality Patterns

| Period | Revenue Impact | Notes |
|--------|---------------|-------|
| Q1 (Jan-Mar) | +5-10% | Flu season, annual checkups |
| Q2 (Apr-Jun) | Baseline | Steady demand |
| Q3 (Jul-Aug) | -10-15% | Summer slowdown, vacation |
| Q4 (Sep-Dec) | +5-10% | Pre-holiday screenings, open enrollment |

---

## 8. Narrative Context for Prompts

```
Medical diagnostics company providing laboratory and imaging services to healthcare
providers across the DACH region. Focus on operational efficiency, regulatory
compliance (ISO 15189, CLIA), turnaround time optimization, and strategic
partnerships with hospital networks. Key metrics include test volume, average
revenue per test, equipment utilization rates, and quality scores.
```

---

## 9. Implementation Checklist

### Files to Modify

| File | Changes |
|------|---------|
| `lib/config/scenario-matrix.ts` | Add `healthcare` to `Sector` type, add `SECTOR_CONFIGS['healthcare']` |
| `lib/config/scenario-matrix.ts` | Add company names for `midcap-healthcare` |
| `scripts/generate-curated-examples.ts` | Replace `midcap-financials` with `midcap-healthcare` |
| `components/landing/ExampleCards.tsx` | Update tile for Healthcare example |
| `lib/prompts/world-building-prompt.ts` | Add healthcare narrative context (if sector-specific) |

### New Files (If Needed)

| File | Purpose |
|------|---------|
| `public/examples/midcap-healthcare/` | Curated example JSON data |

### Tier Gating

- Healthcare available to: **Lifetime+**, **Pro**, **Team** (same as HR domain)
- Free tier sees curated Healthcare example but cannot generate new Healthcare scenarios

---

## 10. Validation Criteria

Before production deployment, validate:

1. [ ] Financial ratios are realistic (compare to industry benchmarks)
2. [ ] HR roles match clinical setting expectations
3. [ ] Department distribution is realistic
4. [ ] Salary ranges align with DACH healthcare market
5. [ ] Revenue events make sense for diagnostics company
6. [ ] Cost structure reflects high-labor, high-equipment reality
7. [ ] No obviously "fake" patterns visible to healthcare professionals

---

## References

- NYU Stern Industry Margins (Healthcare Services)
- McKinsey Healthcare Provider Benchmarks
- Bureau of Labor Statistics - Healthcare Occupations
- ISO 15189 Laboratory Accreditation Standards

---

**Ready for Winston's architecture review.**
