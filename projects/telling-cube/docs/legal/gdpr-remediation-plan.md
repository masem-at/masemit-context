# GDPR Remediation Plan - tellingCube

**Date:** 2025-12-16
**Data Controller:** masemIT e.U. (Austria)
**Based on:** gdpr-gap-analysis.md
**Status:** DRAFT - Requires legal review before implementation

---

## Executive Summary

This remediation plan addresses the gaps identified in the GDPR Gap Analysis. The plan is organized by priority and estimated effort, focusing on practical steps a solo founder can implement without external legal counsel for most items.

**Total Estimated Effort:** 8-12 hours
**Recommended Timeline:** Complete HIGH priority within 2 weeks, MEDIUM within 30 days

---

## Immediate Actions (Do Now)

### 1. Sign Data Processing Agreements (DPAs)
**Priority:** 🔴 HIGH | **Effort:** 1-2 hours | **Risk if Skipped:** Regulatory fine

Most providers offer self-service DPA signing. Complete these in one session:

| Processor | Where to Sign | Link |
|-----------|---------------|------|
| **Stripe** | Dashboard → Settings → Compliance → DPA | [stripe.com/legal/dpa](https://stripe.com/legal/dpa) |
| **Vercel** | Dashboard → Settings → Legal → DPA | [vercel.com/legal/dpa](https://vercel.com/legal/dpa) |
| **NeonDB** | Dashboard → Settings → Legal | [neon.tech/legal](https://neon.tech/legal) |
| **Resend** | Dashboard → Settings → DPA | [resend.com/legal/dpa](https://resend.com/legal/dpa) |
| **Anthropic** | Included in API Terms | [anthropic.com/legal](https://www.anthropic.com/legal) |
| **Cloudflare** | Dashboard → Account → DPA | [cloudflare.com/cloudflare-customer-dpa](https://www.cloudflare.com/cloudflare-customer-dpa/) |
| **Upstash** | Dashboard → Settings → DPA | [upstash.com/trust/dpa](https://upstash.com/trust/dpa) |

**Action Items:**
- [ ] Log into each provider dashboard
- [ ] Locate DPA/GDPR section
- [ ] Sign/accept DPA (usually click-through)
- [ ] Download PDF copy for records
- [ ] Store in `docs/legal/dpas/` folder

**Tip:** Create a folder `docs/legal/dpas/` and save PDF copies of each signed DPA for your records.

---

### 2. Update Privacy Policy
**Priority:** 🔴 HIGH | **Effort:** 2-3 hours | **Risk if Skipped:** Regulatory fine, user complaints

**Required Additions:**

#### A. Add Legal Basis Section
Insert after "How We Use Your Data":

```
## Legal Basis for Processing

We process your data based on the following legal grounds:

| Processing Activity | Legal Basis | GDPR Article |
|---------------------|-------------|--------------|
| Payment processing | Performance of contract | Art. 6(1)(b) |
| Email confirmation | Performance of contract | Art. 6(1)(b) |
| Contact form | Your consent | Art. 6(1)(a) |
| Session cookies | Legitimate interest (essential functionality) | Art. 6(1)(f) |
| Rate limiting | Legitimate interest (security) | Art. 6(1)(f) |
| Scenario generation | Performance of contract | Art. 6(1)(b) |
```

#### B. Add International Transfers Section
Insert after "Third-Party Services":

```
## International Data Transfers

Some of our service providers process data outside the European Economic Area (EEA):

| Provider | Location | Safeguard |
|----------|----------|-----------|
| Anthropic | USA | EU-US Data Privacy Framework / Standard Contractual Clauses |
| Resend | USA | Standard Contractual Clauses (included in DPA) |

Our primary infrastructure (Vercel, NeonDB) is hosted in Frankfurt, Germany (EU).

All international transfers are protected by appropriate safeguards including Standard Contractual Clauses (SCCs) as required by GDPR Article 46.
```

#### C. Complete GDPR Rights Section
Replace current rights section with:

```
## Your Rights (GDPR)

Under the General Data Protection Regulation, you have the following rights:

- **Right of Access** (Art. 15) - Request a copy of your personal data
- **Right to Rectification** (Art. 16) - Correct inaccurate personal data
- **Right to Erasure** (Art. 17) - Request deletion of your data
- **Right to Restrict Processing** (Art. 18) - Limit how we use your data
- **Right to Data Portability** (Art. 19) - Receive your data in a portable format
- **Right to Object** (Art. 21) - Object to processing based on legitimate interest
- **Right to Withdraw Consent** - Where processing is based on consent

**How to Exercise Your Rights:**
Contact us at support@masem.at. We will respond within 30 days.

**Right to Lodge a Complaint:**
If you believe your data protection rights have been violated, you have the right to lodge a complaint with your local supervisory authority. For Austria:

**Österreichische Datenschutzbehörde (DSB)**
Barichgasse 40-42
1030 Vienna, Austria
Email: dsb@dsb.gv.at
Website: https://www.dsb.gv.at
```

#### D. Add Automated Decision-Making Statement
Insert after rights section:

```
## Automated Decision-Making

We do not use automated decision-making or profiling that produces legal effects or similarly significantly affects you.
```

#### E. Add Upstash to Processor List
Add to Third-Party Services:

```
- **Upstash:** Rate limiting and security (see [Upstash Privacy Policy](https://upstash.com/trust/privacy-policy))
```

---

### 3. Fix Imprint (ECG §5 Compliance)
**Priority:** 🟡 LOW | **Effort:** 10 minutes | **Risk if Skipped:** Minor Austrian fine

Add business address to Imprint page:

```
masemIT e.U.
[Your Street Address]
[Postal Code] [City]
Austria

UID: ATU77051468
```

**Note:** Austrian E-Commerce law (ECG §5) requires a geographic address, not just email.

---

## Short-term Actions (Next 30 Days)

### 4. Create Breach Notification Procedure
**Priority:** 🟡 MEDIUM | **Effort:** 2-3 hours | **Risk if Skipped:** Failure to report breach in 72 hours

Create `docs/legal/breach-response-procedure.md`:

```markdown
# Data Breach Response Procedure

## 1. Detection
- Monitor Sentry alerts for security anomalies
- Review access logs weekly
- Respond to user reports of suspicious activity

## 2. Initial Assessment (Within 4 Hours)
- [ ] What data was affected?
- [ ] How many individuals affected?
- [ ] What is the likely risk to individuals?
- [ ] Is the breach ongoing?

## 3. Containment
- [ ] Revoke compromised credentials
- [ ] Patch vulnerability if applicable
- [ ] Preserve evidence for investigation

## 4. Risk Assessment
**High Risk Indicators:**
- Financial data exposed
- Large number of individuals
- Sensitive data categories
- Data likely to be misused

## 5. Notification Decisions

### Supervisory Authority (DSB Austria)
**When:** Within 72 hours if risk to individuals
**How:** dsb@dsb.gv.at or online form at dsb.gv.at
**What to Include:**
- Nature of breach
- Categories and approximate number of individuals
- Name and contact of DPO/contact point
- Likely consequences
- Measures taken or proposed

### Affected Individuals
**When:** Without undue delay if HIGH risk
**How:** Direct email to affected users
**What to Include:**
- Plain language description
- Likely consequences
- Measures taken
- Recommendations for self-protection

## 6. Documentation
Document ALL breaches (even if not notified):
- Date/time of breach
- Date/time of discovery
- Nature of breach
- Data affected
- Individuals affected
- Risk assessment
- Actions taken
- Notification decisions and rationale

## 7. Post-Incident Review
- Root cause analysis
- Process improvements
- Update security measures
```

---

### 5. Create Record of Processing Activities (ROPA)
**Priority:** 🟡 MEDIUM | **Effort:** 3-4 hours | **Risk if Skipped:** Documentation non-compliance

See Step 4 of this workflow for the ROPA template.

---

### 6. Document DSR Handling Procedure
**Priority:** 🟡 MEDIUM | **Effort:** 1-2 hours | **Risk if Skipped:** Failure to respond in 30 days

Create `docs/legal/dsr-procedure.md`:

```markdown
# Data Subject Request (DSR) Procedure

## Receiving Requests
- Monitor support@masem.at for DSR keywords
- Keywords: "my data", "delete my", "access request", "GDPR", "data protection"

## Verification
1. Confirm requester identity (email match)
2. For ambiguous requests, ask for clarification

## Response Timeline
- **Acknowledge:** Within 3 business days
- **Complete:** Within 30 days (extendable to 90 days for complex requests)

## Request Types

### Access Request (Art. 15)
1. Search database for email address
2. Search Stripe for payment records
3. Compile data into readable format
4. Send via secure method

### Deletion Request (Art. 17)
1. Delete from database (contact forms, if any scenarios)
2. Note: Cannot delete payment records (legal obligation)
3. Request Stripe deletion if applicable
4. Confirm deletion to user

### Portability Request (Art. 20)
1. Export data in JSON/CSV format
2. Send via secure method

## Response Templates

### Acknowledgment
Subject: Your Data Request - Received

Dear [Name],

Thank you for your data protection request. We have received your request on [date] and will respond within 30 days.

If we need any clarification, we will contact you.

Best regards,
masemIT e.U.

### Completion
Subject: Your Data Request - Complete

Dear [Name],

We have completed your [access/deletion/portability] request.

[Details of action taken]

If you have any questions, please contact us.

Best regards,
masemIT e.U.
```

---

## Medium-term Actions (Next 90 Days)

### 7. Consider Cookie Consent Banner
**Priority:** 🟢 LOW | **Effort:** 2-4 hours | **Risk if Skipped:** Minimal (session cookies only)

**Current State:** You only use essential session cookies - no tracking.

**Assessment:** A cookie banner is likely NOT required since:
- Session cookies are "strictly necessary"
- No analytics tracking (Plausible is privacy-friendly)
- No advertising cookies

**Optional Enhancement:** Add a simple cookie notice (not consent banner) explaining session cookie use. This is good practice but not legally required for essential cookies.

---

### 8. Annual Compliance Review
**Priority:** 🟢 LOW | **Effort:** 2-3 hours annually

Set calendar reminder for December 2026:
- [ ] Review and update Privacy Policy
- [ ] Verify all DPAs still valid
- [ ] Update ROPA if processing changed
- [ ] Review security measures
- [ ] Check for regulatory changes

---

## Resource Needs

### Internal Effort Required
| Task | Hours | Who |
|------|-------|-----|
| Sign DPAs | 1-2h | Mario |
| Update Privacy Policy | 2-3h | Mario |
| Create Breach Procedure | 2-3h | Mario |
| Create ROPA | 3-4h | Mario (or use Step 4 template) |
| Create DSR Procedure | 1-2h | Mario |
| Fix Imprint | 0.5h | Mario |
| **Total** | **10-15h** | |

### External Help Needed
| Task | When | Estimated Cost |
|------|------|----------------|
| Legal review of Privacy Policy | Before major scaling | €200-500 |
| Full GDPR audit | If enterprise customers | €1,000-3,000 |
| DPO consultation | If required (>250 employees or high-risk processing) | N/A for now |

### Tools/Platforms to Consider
| Tool | Purpose | Cost |
|------|---------|------|
| Termly / iubenda | Privacy policy generator | €10-50/month |
| OneTrust (lite) | Cookie consent management | Free tier available |
| Notion / Coda | Documentation & ROPA tracking | Free |

**Recommendation:** For a solo founder, manual documentation is sufficient. Tools add value at scale.

---

## Implementation Checklist

### Week 1 (Immediate)
- [ ] Sign all 7 DPAs (1-2 hours)
- [ ] Create `docs/legal/dpas/` folder and save copies
- [ ] Fix Imprint with business address (10 min)

### Week 2 (High Priority)
- [ ] Update Privacy Policy with legal basis section
- [ ] Add international transfers section
- [ ] Complete GDPR rights section
- [ ] Add Upstash to processor list
- [ ] Add automated decision-making statement

### Week 3-4 (Medium Priority)
- [ ] Create breach notification procedure
- [ ] Create DSR handling procedure
- [ ] Complete ROPA (see Step 4)

### Month 2-3 (Low Priority)
- [ ] Consider cookie notice (optional)
- [ ] Set up annual review reminder
- [ ] Optional: Legal review of updated documents

---

## Success Metrics

After completing this remediation:

| Metric | Before | After |
|--------|--------|-------|
| DPAs signed | 0/7 | 7/7 |
| Privacy Policy completeness | ~70% | ~95% |
| Documented procedures | 0/3 | 3/3 |
| ROPA exists | No | Yes |
| Overall compliance estimate | 65-70% | 90%+ |

---

## Disclaimer

This remediation plan is a template for guidance purposes only. It does not constitute legal advice. Implementation should be reviewed by a qualified Austrian/EU data protection lawyer, particularly before:
- Processing special category data
- Significant scaling of user base
- Enterprise customer onboarding
- Any regulatory inquiry

---

**Document Version:** 1.0
**Review Date:** After implementation complete
