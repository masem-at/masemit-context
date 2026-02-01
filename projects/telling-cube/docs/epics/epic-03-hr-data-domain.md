# Epic 3: HR Data Domain

**Created:** 2026-01-18
**Status:** Ready for Development
**Sprint:** 3
**Priority:** High
**Estimated Effort:** 2-3 weeks
**Dependencies:** Epic 2 (Multi-Year) - HR events span all generated years

---

## Goal

Add a dedicated HR (Human Resources) data domain to tellingCube, enabling users to generate and export realistic employee-related data including headcount trends, turnover, compensation, org structure, and workforce analytics.

## Business Value

- **Brian's Request:** "Building out an HR view and good HR data is just really difficult to get."
- **Jürgen's Request:** "weitere Domains wie HR und ESG"
- **Market Gap:** No good synthetic HR data available
- **Privacy Solution:** Real HR data is highly sensitive (GDPR) - synthetic solves this

## Source Spec

- `docs/future/spec-hr-data-domain.md`

---

## Stories

### Story 3.1: HR Event Types & Employee Entity
**Points:** 5

**Description:**
Extend data model with HR-specific event types and employee entity in ScenarioWorld.

**Acceptance Criteria:**
- [ ] New event types implemented:
  - `EMPLOYEE_HIRED`
  - `EMPLOYEE_TERMINATED`
  - `EMPLOYEE_PROMOTED`
  - `EMPLOYEE_TRANSFERRED`
  - `SALARY_CHANGE`
  - `TRAINING_COMPLETED`
  - `PERFORMANCE_REVIEW`
- [ ] Employee entity added to world generation:
  - id, firstName, lastName (synthetic)
  - department, costCenter, region
  - role, level (Junior/Mid/Senior/Lead/Manager/Director)
  - hireDate, terminationDate
  - salary, currency
  - birthYear (not full DOB - privacy)
  - gender (M/F/D)
  - skills[], reportsTo
- [ ] Database schema updated

---

### Story 3.2: HR Event Generation Logic
**Points:** 8

**Description:**
Implement AI generation for realistic HR events based on company size/sector.

**Acceptance Criteria:**
- [ ] Turnover rates by company size:
  - Startup: 15-25%, rapid hiring phases
  - MidCap: 10-15%, steady growth
  - LargeCap: 5-10%, large workforce
- [ ] Consistency rules enforced:
  - `SUM(Payroll)` = `COUNT(Active) × Avg Salary`
  - `Hires - Terminations` = `Headcount Change`
  - Salary changes reflected in payroll
- [ ] Multi-Year integration: HR events span all years if Epic 2 active
- [ ] Seasonal patterns (hiring cycles, year-end reviews)

---

### Story 3.3: HR Export API Endpoints
**Points:** 3

**Description:**
Create read-only API endpoints for HR data export.

**Acceptance Criteria:**
- [ ] `GET /api/export/{scenarioId}/hr.json` - Full HR data
- [ ] `GET /api/export/{scenarioId}/hr.csv` - Tabular export
- [ ] Export includes:
  - summary (headcount, avgTenure, turnoverRate, avgSalary, totalPayroll)
  - headcountByDepartment
  - headcountByMonth
  - turnoverByMonth
  - salaryDistribution
  - employees (anonymized list)
- [ ] Proper caching headers

---

### Story 3.4: HR View Page
**Points:** 3

**Description:**
Create new HR view page at `/scenarios/{id}/hr`.

**Acceptance Criteria:**
- [ ] Route `/scenarios/{id}/hr` accessible
- [ ] Tab navigation: Finance | Sales | HR
- [ ] HR summary cards (headcount, turnover %, avg salary, payroll cost)
- [ ] Responsive layout
- [ ] "Synthetic Data" badge visible

---

### Story 3.5: Headcount by Department Chart
**Points:** 3

**Description:**
Create horizontal bar chart showing FTE count per department.

**Acceptance Criteria:**
- [ ] Horizontal bars (IBCS style)
- [ ] Sorted by headcount (largest first)
- [ ] Department labels clear
- [ ] AC vs PL comparison if available
- [ ] Click to filter by department

---

### Story 3.6: Turnover Trend Chart
**Points:** 3

**Description:**
Create chart showing hires, terminations, and net change over time.

**Acceptance Criteria:**
- [ ] Combined chart: Hires (positive), Terminations (negative)
- [ ] Net change line overlay
- [ ] Monthly granularity
- [ ] Turnover rate % annotation
- [ ] Target turnover line if applicable

---

### Story 3.7: Compensation Distribution Chart
**Points:** 3

**Description:**
Create chart showing salary distribution and payroll trends.

**Acceptance Criteria:**
- [ ] Salary band histogram
- [ ] Avg salary by department bars
- [ ] Payroll cost trend line
- [ ] Cost per FTE KPI card
- [ ] Currency formatting correct

---

### Story 3.8: Workforce Demographics Charts
**Points:** 3

**Description:**
Create charts for age distribution, gender ratio, tenure.

**Acceptance Criteria:**
- [ ] Age bracket histogram (20-29, 30-39, etc.)
- [ ] Gender ratio visualization (M/F/D)
- [ ] Tenure distribution chart
- [ ] All charts IBCS-styled
- [ ] Privacy disclaimer visible

---

### Story 3.9: Privacy Disclaimer & Synthetic Data Badge
**Points:** 2

**Description:**
Add clear indicators that HR data is synthetic and GDPR-compliant.

**Acceptance Criteria:**
- [ ] "Synthetic Data" badge on HR view
- [ ] Footer disclaimer: "This data is entirely synthetic and does not represent real individuals"
- [ ] Option to use anonymized names (Employee_001, etc.)
- [ ] No real birthdates exposed (only birth year)
- [ ] Export includes synthetic data notice

---

## Definition of Done

- [ ] All stories completed and tested
- [ ] HR view shows realistic workforce data
- [ ] All charts render correctly
- [ ] Export includes complete HR data
- [ ] Privacy disclaimers in place
- [ ] HR events consistent with Finance (payroll reconciles)
- [ ] Deployed to develop branch for testing

---

## Technical Notes (from Winston)

**Architecture Decision:** HR Domain is a vertical extension (new data domain). It leverages Multi-Year infrastructure from Epic 2 - if a 5-year scenario is generated, HR events span all 5 years with realistic workforce evolution.

**Consistency Rule:** Finance's payroll expenses MUST match HR's total compensation. This is enforced at generation time.

**GDPR Note:** All employee data is synthetic. No real personal data. Birth year only (not DOB). Names from faker.js.

---

*Epic created by: Bob (Scrum Master)*
*Reviewed by: John (PM), Winston (Architect)*
*Source spec by: Mary (Analyst), Winston (Architect)*
