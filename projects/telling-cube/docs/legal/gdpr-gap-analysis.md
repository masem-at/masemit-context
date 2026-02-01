# GDPR Gap Analysis - tellingCube

**Date:** 2025-12-16
**Data Controller:** masemIT e.U. (Austria, UID: ATU77051468)
**Product:** tellingCube (tellingcube.com)
**Prepared by:** Daniel Semper (GDPR & Legal Specialist - AI Agent)
**Status:** DRAFT - Requires legal review before implementation

---

## Executive Summary

tellingCube demonstrates strong **Privacy by Design** principles with its ephemeral data model and minimal data collection. The EU-based controller (Austria) and EU data residency (Frankfurt) significantly simplify compliance. However, several documentation and procedural gaps need addressing before scaling.

**Overall Compliance Rating:** 🟡 **PARTIAL** (estimated 65-70% compliant)

| Category | Rating | Priority |
|----------|--------|----------|
| Data Minimization | ✅ Compliant | - |
| Privacy by Design | ✅ Compliant | - |
| Privacy Policy | 🟡 Partial | High |
| Legal Basis Documentation | 🔴 Gap | High |
| Data Processing Agreements | 🔴 Gap | High |
| Data Subject Rights Process | 🟡 Partial | Medium |
| Security Measures | ✅ Compliant | - |
| Breach Notification | 🔴 Gap | Medium |
| ROPA | 🔴 Gap | Medium |
| International Transfers | 🟡 Partial | Medium |
| Imprint Compliance (ECG §5) | 🟡 Partial | Low |

---

## Compliance Scorecard

### 1. Lawfulness of Processing
**Rating:** 🟡 Partial

**Current State:**
- Processing activities are legitimate and proportionate
- Legal bases exist but are not explicitly documented

**Gap:** No documented legal basis analysis for each processing activity

**Required Legal Bases:**
| Processing Activity | Suggested Legal Basis | GDPR Article |
|---------------------|----------------------|--------------|
| Payment processing (email to Stripe) | Contract performance | Art. 6(1)(b) |
| Email confirmation (Resend) | Contract performance | Art. 6(1)(b) |
| Contact form processing | Consent / Legitimate interest | Art. 6(1)(a) or (f) |
| Session cookies | Legitimate interest (essential) | Art. 6(1)(f) |
| Rate limiting (IP storage) | Legitimate interest (security) | Art. 6(1)(f) |
| Scenario generation | Contract performance | Art. 6(1)(b) |

---

### 2. Transparency
**Rating:** 🟡 Partial

**Current State:**
- Privacy Policy exists ✅
- Third-party processors listed ✅
- Data Controller identified ✅
- Contact information provided ✅

**Gaps in Privacy Policy:**
1. **Missing legal basis per activity** - GDPR Art. 13(1)(c) requires stating legal basis
2. **Incomplete rights list** - Missing: right to rectification, right to object, right to restrict processing, right to lodge complaint with supervisory authority (DSB Austria)
3. **Missing international transfer disclosure** - Anthropic (US), Resend (US) process data outside EEA
4. **Missing Upstash** from processor list - Stores IP addresses for rate limiting
5. **No automated decision-making statement** - Even negative statement ("we don't use automated decision-making") is good practice

---

### 3. Data Minimization
**Rating:** ✅ Compliant

**Strengths:**
- Ephemeral sessions (no user accounts in MVP)
- 24-hour automatic data deletion
- Only essential data collected (email for payment, contact form data)
- Payment data delegated to Stripe (not stored locally)
- Session cookies only (no tracking/analytics cookies)

**Assessment:** Excellent - this is textbook Privacy by Design

---

### 4. Purpose Limitation
**Rating:** ✅ Compliant

**Current State:**
- Data used only for stated purposes
- No secondary use or sharing beyond service delivery
- Clear purpose statements in Privacy Policy

---

### 5. Storage Limitation
**Rating:** ✅ Compliant

**Current State:**
- Scenarios: 24 hours ✅
- Contact forms: 12 months ✅
- Payment records: 7-10 years (tax law requirement) ✅
- Session cookies: Browser session only ✅

**Assessment:** Retention periods are documented and appropriate

---

### 6. Data Subject Rights
**Rating:** 🟡 Partial

**Current State:**
- Rights mentioned in Privacy Policy ✅
- Contact email provided (support@masem.at) ✅

**Gaps:**
1. **Incomplete rights enumeration** - Privacy Policy lists 4 rights, GDPR provides 8
2. **No documented procedure** - How do you handle a DSR request?
3. **No response timeline** - GDPR requires response within 1 month
4. **Missing supervisory authority reference** - Austrian DPA (Datenschutzbehörde)

**Missing from Privacy Policy:**
- Right to rectification (Art. 16)
- Right to restriction of processing (Art. 18)
- Right to object (Art. 21)
- Right to lodge complaint with supervisory authority

---

### 7. International Transfers
**Rating:** 🟡 Partial

**Current State:**
- Primary infrastructure in EU (Vercel Frankfurt, NeonDB Frankfurt) ✅
- Some US processors used

**Transfer Analysis:**
| Processor | Location | Transfer Mechanism | Status |
|-----------|----------|-------------------|--------|
| Vercel | Frankfurt (EU) | N/A - EU | ✅ OK |
| NeonDB | Frankfurt (EU) | N/A - EU | ✅ OK |
| Stripe | EU entity available | SCCs in Stripe DPA | ⚠️ Need DPA |
| Anthropic | US | EU-US DPF / SCCs | ⚠️ Need DPA |
| Resend | US | SCCs | ⚠️ Need DPA |
| Cloudflare | Global/US | SCCs in DPA | ⚠️ Need DPA |
| Upstash | EU available | SCCs | ⚠️ Need DPA |

**Gap:** No documented transfer safeguards or DPAs in place

---

### 8. Processor Agreements (DPAs)
**Rating:** 🔴 Non-compliant

**Current State:** No DPAs signed with any processor

**Required DPAs:**
| Processor | Personal Data Processed | DPA Available | Priority |
|-----------|------------------------|---------------|----------|
| Stripe | Email, payment metadata | Yes (online) | HIGH |
| Vercel | IP addresses, logs | Yes (online) | HIGH |
| NeonDB | Contact form data, emails | Yes (online) | HIGH |
| Anthropic | Scenario prompts (may include context) | Yes (Terms include DPA) | HIGH |
| Resend | Email addresses | Yes (online) | HIGH |
| Cloudflare | IP addresses (Turnstile) | Yes (online) | MEDIUM |
| Upstash | IP addresses (rate limiting) | Yes (online) | MEDIUM |

**Note:** Most major providers have self-service DPA signing. This is typically a click-through process in their dashboards.

---

### 9. Breach Notification Procedures
**Rating:** 🔴 Non-compliant

**Current State:** No documented breach response plan

**GDPR Requirements:**
- Notify supervisory authority within 72 hours (Art. 33)
- Notify affected individuals if high risk (Art. 34)
- Document all breaches (even if not notified)

**Gap:** No procedure for detecting, assessing, or reporting breaches

---

### 10. Documentation (ROPA)
**Rating:** 🔴 Non-compliant

**Current State:** No Record of Processing Activities

**GDPR Requirement:** Art. 30 requires controllers to maintain records of processing activities

**Note:** While there's an exemption for organizations with <250 employees, it doesn't apply if processing:
- Is not occasional, OR
- Includes special categories of data, OR
- Could result in risk to rights and freedoms

tellingCube's regular payment processing likely makes ROPA required.

---

### 11. Security Measures
**Rating:** ✅ Compliant

**Documented Measures:**
- TLS/SSL encryption in transit ✅
- Database encryption at rest (NeonDB) ✅
- Rate limiting (abuse prevention) ✅
- Input validation ✅
- PCI compliance delegated to Stripe ✅
- Security architecture documented ✅

**Assessment:** Strong security foundation

---

### 12. Privacy by Design
**Rating:** ✅ Compliant

**Implemented Principles:**
- Data minimization (ephemeral model) ✅
- Purpose limitation (clear, limited purposes) ✅
- Storage limitation (24-hour cleanup) ✅
- Security by default (HTTPS, encryption) ✅
- No tracking cookies ✅

**Assessment:** Excellent - this architecture demonstrates genuine commitment to privacy

---

### 13. Imprint Compliance (Austrian ECG §5)
**Rating:** 🟡 Partial

**Current State:**
- Company name ✅
- UID number ✅
- Contact email ✅
- Responsible person ✅

**Gaps:**
- **Missing business address** - ECG §5(1) requires geographic address
- **Missing WKO info** - If applicable, chamber membership should be listed

---

## Critical Gaps Summary

### HIGH Priority (Address Before Scaling)

| Gap | Risk | Effort | Article |
|-----|------|--------|---------|
| No DPAs with processors | Regulatory fine, processor liability | Low (1-2 hours) | Art. 28 |
| Legal basis not documented | Regulatory fine | Low (1 hour) | Art. 6, 13 |
| Incomplete Privacy Policy | Regulatory fine, user complaints | Medium (2-3 hours) | Art. 13, 14 |

### MEDIUM Priority (Address Within 30 Days)

| Gap | Risk | Effort | Article |
|-----|------|--------|---------|
| No breach notification procedure | Failure to report breach | Medium (2-3 hours) | Art. 33, 34 |
| No ROPA | Documentation non-compliance | Medium (3-4 hours) | Art. 30 |
| DSR procedure undocumented | Failure to respond in time | Low (1-2 hours) | Art. 15-22 |
| International transfer documentation | Transfer without safeguards | Low (1 hour) | Art. 44-49 |

### LOW Priority (Address Within 90 Days)

| Gap | Risk | Effort | Article |
|-----|------|--------|---------|
| Imprint missing address | ECG §5 fine (minor) | Low (10 min) | ECG §5 |
| Cookie policy (separate page) | Best practice, not strictly required | Low (1 hour) | ePrivacy |

---

## Risk Assessment

### Likelihood of Regulatory Action: LOW

**Mitigating Factors:**
- Small-scale processing
- EU-based controller
- Privacy-friendly architecture
- No sensitive data categories
- B2B focus (fewer individual complaints)

### Potential Impact if Non-Compliant: MEDIUM

**Worst Case Scenarios:**
- DSB (Austria) inquiry: Administrative burden, potential fine up to €20M or 4% revenue
- User complaint: Reputation damage, required remediation
- Data breach without procedure: Enhanced penalties, public notification

### Overall Risk Level: 🟡 MODERATE

The combination of low likelihood + medium impact = manageable risk, but gaps should be addressed proactively.

---

## Positive Findings

1. **EU Data Residency** - Frankfurt hosting eliminates most transfer concerns
2. **Austrian Controller** - EU-based simplifies compliance significantly
3. **Ephemeral Model** - 24-hour data deletion is excellent privacy by design
4. **Minimal Data Collection** - Only essential data collected
5. **No Tracking** - Session cookies only, no analytics tracking
6. **Security Documentation** - Comprehensive security architecture exists
7. **Existing Legal Pages** - Privacy Policy, Terms, Imprint already in place
8. **Clear Contact Points** - support@masem.at for privacy inquiries

---

## Disclaimer

This gap analysis is a template for discussion purposes only. It does not constitute legal advice. Before implementing any changes to legal documents or compliance processes, please consult with a qualified Austrian/EU data protection lawyer.

---

**Document Version:** 1.0
**Next Review:** After remediation implementation
