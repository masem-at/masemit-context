# Feature Spec: HR Data Domain

**Created:** 2026-01-16
**Status:** Draft for Review
**Requested By:** Brian Julius (video feedback), Jürgen Faisst (email feedback)
**Priority:** High

---

## Executive Summary

Add a dedicated HR (Human Resources) data domain to tellingCube, enabling users to generate and export realistic employee-related data including headcount trends, turnover, compensation, org structure, and workforce analytics.

**Brian's Quote:**
> *"One of the things I love about this is building out an HR view and good HR data is just really difficult to get."*

**Jürgen's Quote:**
> *"weitere Domains wie HR und ESG"*

---

## Problem Statement

### Current State
- tellingCube generates Finance and Sales data
- Employee data exists in events (payroll) but is not exposed as a dedicated view
- HR professionals and BI developers lack realistic HR test data
- Real HR data is highly sensitive (GDPR, personal data)

### Pain Points
1. **HR Analytics Testing:** No good synthetic HR data available
2. **Dashboard Development:** Can't test HR dashboards without realistic data
3. **Training:** HR analysts need practice data for learning tools
4. **Privacy:** Real HR data cannot be shared or used for demos

---

## Proposed Solution

### HR View (New Export Option)

Add `/hr` view alongside existing `/finance` and `/sales` views.

#### HR Metrics to Generate

**1. Headcount & Structure**
| Metric | Description | IBCS Visual |
|--------|-------------|-------------|
| Headcount by Department | FTE count per cost center | Horizontal bars |
| Headcount by Region | Geographic distribution | Horizontal bars |
| Headcount Trend | Monthly headcount over time | Line/Area chart |
| Org Hierarchy | Reporting structure | Tree/Sunburst |

**2. Turnover & Movement**
| Metric | Description | IBCS Visual |
|--------|-------------|-------------|
| Hires by Month | New employees per month | Column chart |
| Terminations by Month | Exits per month | Column chart |
| Turnover Rate | Monthly/Annual turnover % | Line + Target |
| Tenure Distribution | Employees by years of service | Histogram |

**3. Compensation**
| Metric | Description | IBCS Visual |
|--------|-------------|-------------|
| Avg Salary by Department | Mean compensation per dept | Horizontal bars |
| Salary Bands | Distribution across bands | Histogram |
| Payroll Cost Trend | Total payroll over time | Area chart |
| Cost per FTE | Avg cost per employee | KPI card |

**4. Workforce Analytics**
| Metric | Description | IBCS Visual |
|--------|-------------|-------------|
| Age Distribution | Employees by age bracket | Histogram |
| Gender Ratio | M/F/Diverse split | Pie/Stacked bar |
| Skills Matrix | Skills by department | Heatmap |
| Training Hours | Hours per employee | Bar chart |

---

## Technical Implementation

### Phase 1: Data Model Extension

**New Event Types:**
```typescript
type HREventType =
  | 'EMPLOYEE_HIRED'
  | 'EMPLOYEE_TERMINATED'
  | 'EMPLOYEE_PROMOTED'
  | 'EMPLOYEE_TRANSFERRED'
  | 'SALARY_CHANGE'
  | 'TRAINING_COMPLETED'
  | 'PERFORMANCE_REVIEW';
```

**Extended Employee Entity (in ScenarioWorld):**
```typescript
interface Employee {
  id: string;
  firstName: string;      // Anonymized/synthetic
  lastName: string;       // Anonymized/synthetic
  department: string;
  costCenter: string;
  region: string;
  role: string;
  level: string;          // Junior/Mid/Senior/Lead/Manager/Director
  hireDate: Date;
  terminationDate?: Date;
  salary: number;
  currency: string;
  birthYear: number;      // For age distribution (not full DOB - privacy)
  gender: 'M' | 'F' | 'D';
  skills: string[];
  reportsTo?: string;     // Manager ID
}
```

### Phase 2: Generation Logic

**HR Event Generation Rules:**
1. Startups: Higher turnover (15-25%), rapid hiring phases
2. MidCaps: Moderate turnover (10-15%), steady growth
3. LargeCaps: Lower turnover (5-10%), large workforce, clear hierarchy

**Consistency Rules:**
- `SUM(Payroll Events)` must match `COUNT(Active Employees) × Avg Salary`
- `Hires - Terminations` must equal `Headcount Change`
- Salary changes must reflect in subsequent payroll events

### Phase 3: Export & API

**New Endpoints:**
```
GET /api/export/{scenarioId}/hr.json
GET /api/export/{scenarioId}/hr.csv
```

**HR Export Structure:**
```json
{
  "summary": {
    "totalHeadcount": 150,
    "avgTenure": 3.2,
    "turnoverRate": 0.12,
    "avgSalary": 65000,
    "totalPayrollCost": 9750000
  },
  "headcountByDepartment": [...],
  "headcountByMonth": [...],
  "turnoverByMonth": [...],
  "salaryDistribution": [...],
  "employees": [...]  // Anonymized employee list
}
```

### Phase 4: UI/Charts

**New HR View Page:** `/scenarios/{id}/hr`

**Chart Components:**
- `HRHeadcountChart.tsx` - Department breakdown
- `HRTurnoverChart.tsx` - Hires/Terminations trend
- `HRSalaryChart.tsx` - Compensation distribution
- `HRDemographicsChart.tsx` - Age/Gender/Tenure

---

## Data Privacy Considerations

### GDPR Compliance (Important!)
- All generated names are **synthetic** (faker.js style)
- No real birthdates - only birth year for age brackets
- No real addresses or contact info
- Data clearly marked as "synthetic/demo data"

### Best Practice
- Include disclaimer: "This data is entirely synthetic and does not represent real individuals"
- Provide option to anonymize even further (Employee_001, Employee_002)

---

## Effort Estimation

| Phase | Description | Complexity |
|-------|-------------|------------|
| Phase 1 | Data model extension | Medium |
| Phase 2 | Generation logic | High |
| Phase 3 | Export & API | Low |
| Phase 4 | UI/Charts | Medium |

**Total Estimated Effort:** 2-3 weeks (solo developer)

---

## Success Metrics

1. **Usage:** HR exports downloaded by >20% of users
2. **Feedback:** Positive mentions of HR data quality
3. **Conversion:** HR domain mentioned in purchase decisions

---

## Open Questions

1. Should HR data be a separate pricing tier or included in all plans?
2. Which HR metrics are most valuable? (User research needed)
3. Include ESG metrics in HR domain or separate?

---

## Next Steps

1. [ ] Mario reviews spec
2. [ ] Prioritize Phase 1-4 sequence
3. [ ] Create user stories in epics
4. [ ] Design HR view mockups
5. [ ] Implement Phase 1 (data model)

---

*Prepared by: Mary (Business Analyst) + Winston (Architect)*
*For review by: Mario*
