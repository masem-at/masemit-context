# Record of Processing Activities (ROPA) - tellingCube

**Data Controller:** masemIT e.U.
**UID:** ATU77051468
**Contact:** support@masem.at
**Date Created:** 2025-12-16
**Last Updated:** 2025-12-16
**Status:** DRAFT - Requires verification and legal review

---

## About This Document

This Record of Processing Activities (ROPA) is maintained in accordance with GDPR Article 30. It documents all processing activities involving personal data conducted by masemIT e.U. in relation to the tellingCube product.

**Review Schedule:** Annually or when processing activities change significantly.

---

## Processing Activity 1: Payment Processing

| Field | Details |
|-------|---------|
| **Activity Name** | Founding Member Payment Processing |
| **Purpose** | Process one-time payments for founding member tiers |
| **Legal Basis** | Art. 6(1)(b) - Performance of contract |
| **Categories of Data Subjects** | Customers (founding members) |
| **Categories of Personal Data** | Email address, payment metadata (amount, date, tier) |
| **Source of Data** | Directly from data subject via Stripe Checkout |
| **Recipients (Internal)** | Mario Semper (sole proprietor) |
| **Recipients (External)** | Stripe (payment processor) |
| **Transfer Outside EEA** | No - Stripe EU entity processes EU payments |
| **Retention Period** | 7 years (Austrian tax law requirement - BAO §132) |
| **Security Measures** | PCI compliance delegated to Stripe; no card data stored locally; HTTPS; access controls |
| **Automated Decision-Making** | No |

---

## Processing Activity 2: Email Confirmation

| Field | Details |
|-------|---------|
| **Activity Name** | Transactional Email Delivery |
| **Purpose** | Send payment confirmation and welcome emails to founding members |
| **Legal Basis** | Art. 6(1)(b) - Performance of contract |
| **Categories of Data Subjects** | Customers (founding members) |
| **Categories of Personal Data** | Email address, purchase tier, purchase date |
| **Source of Data** | Stripe webhook (after successful payment) |
| **Recipients (Internal)** | Mario Semper (sole proprietor) |
| **Recipients (External)** | Resend (email service provider) |
| **Transfer Outside EEA** | Yes - Resend (USA); Safeguard: Standard Contractual Clauses in DPA |
| **Retention Period** | Email logs retained by Resend per their retention policy; local records 7 years |
| **Security Measures** | TLS encryption; API key authentication; no email content stored locally |
| **Automated Decision-Making** | No |

---

## Processing Activity 3: Contact Form Processing

| Field | Details |
|-------|---------|
| **Activity Name** | Contact Form Inquiry Handling |
| **Purpose** | Receive and respond to user inquiries via contact form |
| **Legal Basis** | Art. 6(1)(a) - Consent (checkbox) OR Art. 6(1)(f) - Legitimate interest (pre-contractual inquiries) |
| **Categories of Data Subjects** | Website visitors, potential customers |
| **Categories of Personal Data** | Name, email address, company (optional), message content, selected interests |
| **Source of Data** | Directly from data subject via contact form |
| **Recipients (Internal)** | Mario Semper (sole proprietor) |
| **Recipients (External)** | NeonDB (database storage), Cloudflare Turnstile (bot protection) |
| **Transfer Outside EEA** | Cloudflare (global CDN, USA headquarters); Safeguard: SCCs in DPA |
| **Retention Period** | 12 months, then deleted |
| **Security Measures** | Database encryption at rest; TLS in transit; bot protection; input validation |
| **Automated Decision-Making** | No |

---

## Processing Activity 4: Scenario Generation

| Field | Details |
|-------|---------|
| **Activity Name** | AI-Powered Business Data Generation |
| **Purpose** | Generate synthetic business datasets for users |
| **Legal Basis** | Art. 6(1)(b) - Performance of contract (for members) / Art. 6(1)(f) - Legitimate interest (free trial) |
| **Categories of Data Subjects** | Website users (anonymous session-based) |
| **Categories of Personal Data** | Minimal: Session ID, scenario preferences (no PII in generation prompts) |
| **Source of Data** | User interaction with scenario selection |
| **Recipients (Internal)** | Mario Semper (sole proprietor) |
| **Recipients (External)** | Anthropic (Claude API for AI generation) |
| **Transfer Outside EEA** | Yes - Anthropic (USA); Safeguard: DPF certification + SCCs |
| **Retention Period** | 24 hours (automatic deletion) |
| **Security Measures** | No PII included in prompts; ephemeral storage; TLS encryption |
| **Automated Decision-Making** | AI generation (not decision-making affecting individuals) |

---

## Processing Activity 5: Rate Limiting & Security

| Field | Details |
|-------|---------|
| **Activity Name** | API Rate Limiting and Abuse Prevention |
| **Purpose** | Prevent abuse, ensure fair usage, protect against attacks |
| **Legal Basis** | Art. 6(1)(f) - Legitimate interest (security) |
| **Categories of Data Subjects** | All website visitors and API users |
| **Categories of Personal Data** | IP addresses, request timestamps, request counts |
| **Source of Data** | Automatically collected from HTTP requests |
| **Recipients (Internal)** | Mario Semper (sole proprietor) |
| **Recipients (External)** | Upstash (Redis rate limiting service) |
| **Transfer Outside EEA** | Configurable - EU region available; verify Upstash region setting |
| **Retention Period** | Sliding window (1 hour), then automatically purged |
| **Security Measures** | Encrypted connections; no logging of request content; minimal data |
| **Automated Decision-Making** | Yes - automatic blocking when rate limit exceeded (security measure, not profiling) |

---

## Processing Activity 6: Website Hosting & Logs

| Field | Details |
|-------|---------|
| **Activity Name** | Website Hosting and Server Logs |
| **Purpose** | Host and serve the tellingCube website; maintain security logs |
| **Legal Basis** | Art. 6(1)(f) - Legitimate interest (service delivery, security) |
| **Categories of Data Subjects** | All website visitors |
| **Categories of Personal Data** | IP addresses, browser user agent, access timestamps, pages visited |
| **Source of Data** | Automatically collected by hosting infrastructure |
| **Recipients (Internal)** | Mario Semper (sole proprietor) |
| **Recipients (External)** | Vercel (hosting provider) |
| **Transfer Outside EEA** | No - Vercel Frankfurt (EU) region configured |
| **Retention Period** | Per Vercel retention policy (typically 30 days for logs) |
| **Security Measures** | TLS encryption; DDoS protection; access controls |
| **Automated Decision-Making** | No |

---

## Processing Activity 7: Database Storage

| Field | Details |
|-------|---------|
| **Activity Name** | Application Database |
| **Purpose** | Store contact form submissions, scenario metadata, application data |
| **Legal Basis** | Art. 6(1)(b) - Contract performance / Art. 6(1)(f) - Legitimate interest |
| **Categories of Data Subjects** | Contact form submitters, scenario generators |
| **Categories of Personal Data** | Contact form data (name, email, company, message); scenario metadata |
| **Source of Data** | User submissions and application operations |
| **Recipients (Internal)** | Mario Semper (sole proprietor) |
| **Recipients (External)** | NeonDB (database hosting) |
| **Transfer Outside EEA** | No - NeonDB Frankfurt (EU) region configured |
| **Retention Period** | Contact forms: 12 months; Scenarios: 24 hours; Payment refs: 7 years |
| **Security Measures** | Encryption at rest; TLS in transit; parameterized queries; access controls |
| **Automated Decision-Making** | No |

---

## Summary of Data Processors

| Processor | Role | Location | DPA Status | Transfer Safeguard |
|-----------|------|----------|------------|-------------------|
| Stripe | Payment processing | EU (Ireland) | ☐ Pending | N/A (EU) |
| Vercel | Hosting | EU (Frankfurt) | ☐ Pending | N/A (EU) |
| NeonDB | Database | EU (Frankfurt) | ☐ Pending | N/A (EU) |
| Resend | Email delivery | USA | ☐ Pending | SCCs |
| Anthropic | AI generation | USA | ☐ Pending | DPF + SCCs |
| Cloudflare | Bot protection | Global/USA | ☐ Pending | SCCs |
| Upstash | Rate limiting | EU available | ☐ Pending | SCCs (if US) |

**Action Required:** Update checkboxes to ☑ after signing each DPA.

---

## Summary of Retention Periods

| Data Category | Retention Period | Deletion Method |
|---------------|------------------|-----------------|
| Generated scenarios | 24 hours | Automatic database cleanup |
| Contact form submissions | 12 months | Manual or scheduled deletion |
| Payment records | 7 years | Manual deletion after legal period |
| Server logs (Vercel) | ~30 days | Automatic per provider policy |
| Rate limit data | 1 hour sliding window | Automatic purge |
| Session cookies | Browser session | Automatic on browser close |

---

## Summary of Legal Bases

| Legal Basis | Processing Activities |
|-------------|----------------------|
| **Art. 6(1)(a) - Consent** | Contact form (optional) |
| **Art. 6(1)(b) - Contract** | Payment processing, email confirmation, scenario generation (members) |
| **Art. 6(1)(f) - Legitimate Interest** | Rate limiting, hosting logs, security measures, session cookies |

---

## Data Subject Rights Implementation

| Right | How Exercised | Response Time |
|-------|---------------|---------------|
| Access (Art. 15) | Email to support@masem.at | 30 days |
| Rectification (Art. 16) | Email to support@masem.at | 30 days |
| Erasure (Art. 17) | Email to support@masem.at | 30 days |
| Restriction (Art. 18) | Email to support@masem.at | 30 days |
| Portability (Art. 20) | Email to support@masem.at | 30 days |
| Object (Art. 21) | Email to support@masem.at | 30 days |

**Note:** Payment records cannot be deleted during legal retention period (7 years) per Austrian tax law.

---

## International Transfer Summary

| Destination | Processors | Safeguard Mechanism |
|-------------|------------|---------------------|
| USA | Anthropic, Resend, Cloudflare | Standard Contractual Clauses (SCCs) |
| EU (no transfer) | Stripe, Vercel, NeonDB | N/A - Data remains in EEA |

---

## Security Measures Summary

### Technical Measures
- [x] TLS/SSL encryption for all data in transit
- [x] Database encryption at rest (NeonDB)
- [x] Parameterized queries (SQL injection prevention)
- [x] Input validation (XSS prevention)
- [x] Rate limiting (abuse prevention)
- [x] HTTPS enforced (Vercel)
- [x] Bot protection (Cloudflare Turnstile)

### Organizational Measures
- [x] Minimal data collection policy
- [x] 24-hour ephemeral data deletion
- [x] Single person access (sole proprietor)
- [x] Environment variables for secrets (not in code)
- [ ] Breach notification procedure (pending - see remediation plan)
- [ ] DSR handling procedure (pending - see remediation plan)

---

## Change Log

| Date | Change | Author |
|------|--------|--------|
| 2025-12-16 | Initial ROPA created | Daniel Semper (AI Agent) |
| | | |

---

## Next Review

**Scheduled:** December 2026 or upon significant changes to processing activities.

**Triggers for Earlier Review:**
- New data processor added
- New category of personal data collected
- Significant change in processing purposes
- Security incident
- Regulatory inquiry

---

## Disclaimer

This ROPA is a template created for documentation purposes. It should be:
1. Verified against actual implementation
2. Updated when processing activities change
3. Reviewed by qualified legal counsel before reliance
4. Kept current and accessible for supervisory authority requests

---

**Document Version:** 1.0
**Controller Signature:** _________________ Date: _________
