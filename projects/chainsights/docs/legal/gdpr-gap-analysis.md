# GDPR Gap Analysis: ChainSights

**Date:** 2025-12-16
**Analyst:** Daniel Semper (BMAD Legal)
**Status:** Draft - Requires Legal Review

---

## Executive Summary

ChainSights is a governance analytics service processing personal data from EU customers. The current implementation has **solid foundations** (minimal data collection, no tracking, secure payment handling) but **critical gaps** in data subject rights implementation and documentation.

**Overall Assessment:** Partially Compliant - Remediation Required

**Risk Level:** Medium-High (operating without full compliance exposes regulatory and reputational risk)


---

## Compliance Scorecard

| GDPR Area | Status | Risk | Notes |
|-----------|--------|------|-------|
| Lawfulness of Processing | ✅ Compliant | Low | Contract basis valid for service delivery |
| Transparency | 🟡 Partial | Medium | Privacy policy exists but incomplete |
| Data Minimization | ✅ Compliant | Low | Only essential data collected |
| Purpose Limitation | ✅ Compliant | Low | Data used only for stated purpose |
| Storage Limitation | ❌ Non-compliant | High | No retention policy or deletion mechanism |
| Data Subject Rights | ❌ Non-compliant | High | No access, export, or deletion endpoints |
| Security Measures | 🟡 Partial | Medium | Good basics, some gaps |
| Breach Procedures | ❌ Non-compliant | Medium | No documented breach response plan |
| Processor Management | 🟡 Partial | Medium | Using compliant processors, no DPAs signed |
| Documentation | 🟡 Partial | Medium | No ROPA, no DPIA for AI processing |

---

## Personal Data Inventory

### Data Collected

| Data Type | Legal Basis | Retention | Recipients |
|-----------|-------------|-----------|------------|
| Email address | Contract | Indefinite ⚠️ | Stripe, Resend |
| Customer name | Contract | Indefinite ⚠️ | Stripe, Resend |
| DAO identifier | Contract | Indefinite ⚠️ | Anthropic |
| Payment data | Contract | Handled by Stripe | Stripe only |
| Governance data | Legitimate Interest | Indefinite ⚠️ | Anthropic |
| Admin session | Legitimate Interest | 24 hours ✅ | Internal only |

### Third-Party Processors

| Processor | Data Shared | Location | DPA Status |
|-----------|-------------|----------|------------|
| **Stripe** | Email, name, payment | US (SCCs) | ⚠️ Not signed |
| **Neon** | All database fields | US (SCCs) | ⚠️ Not signed |
| **Resend** | Email, name, PDF report | US (SCCs) | ⚠️ Not signed |
| **Anthropic** | DAO data, governance analysis | US (SCCs) | ⚠️ Not signed |
| **Upstash** | Report ID only | EU/US | ⚠️ Not signed |
| **Vercel** | Application hosting | US (SCCs) | ⚠️ Not signed |

**Note:** All processors have standard DPAs available. You need to formally accept/sign them.

---

## Critical Gaps (High Risk)

### 1. No Data Subject Rights Implementation

**Requirement:** GDPR Articles 15-20 require you to provide data access, rectification, erasure, and portability.

**Current State:** No endpoints or processes exist for:
- Customers to access their data
- Customers to request deletion
- Customers to export their data

**Risk:** Individual complaints to Austrian DPA; fines up to €20M or 4% of turnover.

**Remediation:** Implement `/api/user/data` endpoints + process for manual requests.

---

### 2. No Data Retention Policy

**Requirement:** GDPR Article 5(1)(e) - Data must not be kept longer than necessary.

**Current State:** All data persists indefinitely in database. No cleanup jobs.

**Risk:** Holding unnecessary data increases breach impact and regulatory exposure.

**Remediation:** Define retention periods; implement automatic deletion after period expires.

**Recommended Retention:**
| Data Type | Retention Period | Justification |
|-----------|------------------|---------------|
| Orders | 7 years | Austrian tax/accounting requirements |
| Reports | 1 year after delivery | Service warranty period |
| Raw governance data | 30 days | Only needed for regeneration |
| Share rewards | 1 year | Promotional period |

---

### 3. Incomplete Privacy Policy

**Requirement:** GDPR Articles 13-14 require comprehensive privacy notices.

**Current State:** Privacy policy at `/privacy` exists but is missing:
- Complete list of data recipients (Anthropic, Neon, Upstash not mentioned)
- Specific retention periods
- Data subject rights (access, deletion, portability, objection)
- DPO contact (if applicable) or lead supervisory authority
- Legal basis for each processing activity
- International transfer mechanisms (SCCs)

**Risk:** Non-compliant transparency = invalid consent/contract basis defense.

**Remediation:** Update privacy policy with all required GDPR disclosures.

---

### 4. No Data Processing Agreements

**Requirement:** GDPR Article 28 requires written contracts with all processors.

**Current State:** Using Stripe, Resend, Anthropic, Neon, Vercel, Upstash without formal DPA acceptance.

**Risk:** Joint liability for processor breaches; no contractual protection.

**Remediation:** Accept DPAs from each processor (most have online acceptance):
- Stripe: https://stripe.com/legal/dpa
- Vercel: https://vercel.com/legal/dpa
- Neon: https://neon.tech/legal/dpa
- Resend: https://resend.com/legal/dpa
- Anthropic: https://www.anthropic.com/legal/commercial-terms
- Upstash: https://upstash.com/trust/dpa

---

## Medium Priority Gaps

### 5. AI Processing Without DPIA

**Requirement:** GDPR Article 35 requires Data Protection Impact Assessment for high-risk processing, including automated decision-making.

**Current State:** Anthropic Claude analyzes governance data and produces assessments. No DPIA documented.

**Risk:** Automated analysis that affects how DAOs are perceived could be considered profiling.

**Mitigation:** ChainSights reports are *informational* not *decisional* - they don't trigger automatic consequences. However, documenting this in a lightweight DPIA is recommended.

---

### 6. No Breach Response Plan

**Requirement:** GDPR Article 33-34 requires notification within 72 hours.

**Current State:** No documented breach response procedure.

**Risk:** Delayed response in breach scenario; regulatory penalties.

**Remediation:** Create breach response playbook with:
- Detection procedures
- Assessment criteria
- Notification templates
- Contact list (DPA, affected users)

---

### 7. Admin Authentication Security

**Current State:** Plain-text password comparison in `validateAdminPassword()`.

**Risk:** If `.env` file is exposed, admin access is immediately compromised.

**Remediation:** Hash password with bcrypt; implement rate limiting on login attempts.

---

## Low Priority Gaps

### 8. No Cookie Consent Banner

**Current State:** Only using essential session cookie for admin auth. No tracking.

**Assessment:** Cookie consent banner **not required** for strictly necessary cookies.

**Status:** ✅ Compliant - No action needed unless you add analytics.

---

### 9. No Audit Logging

**Current State:** No logs of who accessed what data when.

**Risk:** Cannot demonstrate accountability or investigate incidents.

**Remediation:** Add access logging for admin panel actions.

---

## What You're Doing Right

| Area | Implementation | Assessment |
|------|----------------|------------|
| **Data Minimization** | Only email, name, DAO address collected | ✅ Excellent |
| **No Tracking** | Zero analytics or marketing pixels | ✅ Excellent |
| **Payment Security** | Stripe handles all card data | ✅ Excellent |
| **Secure Cookies** | HTTP-only, Secure, SameSite | ✅ Good |
| **HTTPS** | All traffic encrypted | ✅ Good |
| **Processor Selection** | All processors are GDPR-compliant | ✅ Good |

---

## Regulatory Context

**Lead Supervisory Authority:** Austrian Data Protection Authority (Datenschutzbehörde)

**Applicable Law:**
- GDPR (Regulation 2016/679)
- Austrian Data Protection Act (DSG)
- Austrian E-Commerce Act (ECG) for imprint requirements

**Business Context:**
- B2B service (DAOs as customers)
- One-time transactions (not ongoing relationship)
- Low data volume (MVP phase)
- No special category data processed

**Risk Assessment:**
- Likelihood of complaint: Low (B2B, professional customers)
- Impact of complaint: Medium (reputational + regulatory)
- Overall risk: Medium - Address critical gaps before scaling

---

## Disclaimer

This gap analysis is provided as a template for internal compliance assessment. It does not constitute legal advice. ChainSights should engage qualified legal counsel to:

1. Review and validate this assessment
2. Finalize privacy policy language
3. Ensure compliance with Austrian-specific requirements
4. Advise on any sector-specific regulations (financial services, if applicable)

---

*Generated by Daniel Semper (BMAD Legal Agent) | 2025-12-16*
