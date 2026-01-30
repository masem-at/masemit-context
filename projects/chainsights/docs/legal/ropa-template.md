# Record of Processing Activities (ROPA)

## Controller Information

| Field | Value |
|-------|-------|
| **Organization** | ChainSights |
| **Address** | Vienna, Austria |
| **Contact** | hello@chainsights.one |
| **Representative** | Mario Semper |
| **DPO** | Not required (SME exemption) |

**Last Updated:** 2025-12-16

---

## Processing Activity 1: Customer Order Processing

| Field | Description |
|-------|-------------|
| **Activity Name** | Customer Order Processing |
| **Purpose** | Process payments and fulfill governance report orders |
| **Legal Basis** | Art. 6(1)(b) - Performance of contract |
| **Categories of Data Subjects** | Customers (DAO governance leads, contributors) |
| **Categories of Personal Data** | Email address, name (optional), payment transaction ID |
| **Source of Data** | Directly from data subject via checkout form |
| **Recipients (Internal)** | Admin (founder) |
| **Recipients (External)** | Stripe (payment processor) |
| **Third Country Transfers** | USA (Stripe) - SCCs in place |
| **Retention Period** | 7 years (Austrian tax requirements) |
| **Security Measures** | HTTPS, encrypted database, Stripe handles payment data |
| **Automated Decision-Making** | None |

---

## Processing Activity 2: Governance Report Generation

| Field | Description |
|-------|-------------|
| **Activity Name** | Governance Report Generation |
| **Purpose** | Generate governance analytics reports for customer DAOs |
| **Legal Basis** | Art. 6(1)(b) - Performance of contract |
| **Categories of Data Subjects** | Customers; DAO voters (public blockchain data) |
| **Categories of Personal Data** | Customer email, DAO identifier, governance data (public wallet addresses, voting records) |
| **Source of Data** | Customer (DAO identifier); Snapshot.org API (public blockchain data) |
| **Recipients (Internal)** | Admin (founder) for quality review |
| **Recipients (External)** | Anthropic (AI analysis), Neon (database storage) |
| **Third Country Transfers** | USA (Anthropic, Neon) - SCCs in place |
| **Retention Period** | 1 year after delivery; raw data 30 days |
| **Security Measures** | HTTPS, encrypted database, API key authentication |
| **Automated Decision-Making** | AI analysis with human review before delivery |

---

## Processing Activity 3: Report Delivery

| Field | Description |
|-------|-------------|
| **Activity Name** | Report Delivery via Email |
| **Purpose** | Deliver completed governance reports to customers |
| **Legal Basis** | Art. 6(1)(b) - Performance of contract |
| **Categories of Data Subjects** | Customers |
| **Categories of Personal Data** | Email address, name, PDF report (contains governance analysis) |
| **Source of Data** | Order processing; Report generation |
| **Recipients (Internal)** | None (automated) |
| **Recipients (External)** | Resend (email delivery service) |
| **Third Country Transfers** | USA (Resend) - SCCs in place |
| **Retention Period** | Email logs retained by Resend per their policy |
| **Security Measures** | TLS encryption, authenticated API |
| **Automated Decision-Making** | None |

---

## Processing Activity 4: Share & Save Rewards

| Field | Description |
|-------|-------------|
| **Activity Name** | Share & Save Cashback Program |
| **Purpose** | Reward customers who share reports publicly |
| **Legal Basis** | Art. 6(1)(b) - Performance of contract (promotional offer) |
| **Categories of Data Subjects** | Customers participating in promotion |
| **Categories of Personal Data** | Twitter handle, tweet URL, follower count, engagement metrics |
| **Source of Data** | Customer submission; public Twitter data |
| **Recipients (Internal)** | Admin (verification) |
| **Recipients (External)** | None |
| **Third Country Transfers** | None |
| **Retention Period** | 1 year |
| **Security Measures** | Database encryption, admin authentication |
| **Automated Decision-Making** | None - manual verification |

---

## Processing Activity 5: Admin Authentication

| Field | Description |
|-------|-------------|
| **Activity Name** | Admin Panel Access Control |
| **Purpose** | Secure access to admin dashboard |
| **Legal Basis** | Art. 6(1)(f) - Legitimate interest (security) |
| **Categories of Data Subjects** | Admin users (internal) |
| **Categories of Personal Data** | Session token (no PII stored) |
| **Source of Data** | Authentication process |
| **Recipients (Internal)** | Admin system |
| **Recipients (External)** | None |
| **Third Country Transfers** | None |
| **Retention Period** | 24 hours (session expiry) |
| **Security Measures** | HTTP-only cookies, secure flag, session expiry |
| **Automated Decision-Making** | None |

---

## Processing Activity 6: Application Hosting & Logs

| Field | Description |
|-------|-------------|
| **Activity Name** | Application Hosting |
| **Purpose** | Host and operate the ChainSights web application |
| **Legal Basis** | Art. 6(1)(f) - Legitimate interest (service operation) |
| **Categories of Data Subjects** | Website visitors, customers |
| **Categories of Personal Data** | IP addresses (in server logs), request metadata |
| **Source of Data** | Automatic collection via web server |
| **Recipients (Internal)** | None (not actively accessed) |
| **Recipients (External)** | Vercel (hosting provider) |
| **Third Country Transfers** | USA (Vercel) - SCCs in place |
| **Retention Period** | Per Vercel retention policy (typically 30 days) |
| **Security Measures** | HTTPS, Vercel security infrastructure |
| **Automated Decision-Making** | None |

---

## Processor Register

| Processor | Processing Activity | Location | DPA Status | Contact |
|-----------|---------------------|----------|------------|---------|
| Stripe, Inc. | Payment processing | USA | ☐ Pending | privacy@stripe.com |
| Neon, Inc. | Database hosting | Germany (EU) | ☐ Pending | privacy@neon.tech |
| Resend, Inc. | Email delivery | Ireland (EU) | ☐ Pending | privacy@resend.com |
| Anthropic PBC | AI analysis | USA | ☐ Pending | privacy@anthropic.com |
| Vercel, Inc. | Application hosting | Germany (EU) | ☐ Pending | privacy@vercel.com |
| Upstash, Inc. | Job queue | USA/EU | ☐ Pending | privacy@upstash.com |

**Update DPA Status to ☑ Complete after signing each agreement.**

---

## International Transfers Summary

All transfers to the United States are protected by:
- **Standard Contractual Clauses (SCCs)** - EU Commission approved
- **Data Processing Agreements** with each processor
- **Supplementary measures** as required post-Schrems II

| Destination | Processors | Transfer Mechanism |
|-------------|------------|-------------------|
| USA | Stripe, Neon, Resend, Anthropic, Vercel, Upstash | SCCs (2021 version) |

---

## Technical & Organizational Measures

### Technical Measures

| Measure | Implementation |
|---------|----------------|
| Encryption in transit | TLS 1.2+ for all connections |
| Encryption at rest | Neon PostgreSQL encrypted storage |
| Access control | Password-protected admin panel |
| Authentication | Session-based with secure cookies |
| Payment security | Delegated to PCI-DSS compliant Stripe |

### Organizational Measures

| Measure | Implementation |
|---------|----------------|
| Access limitation | Single admin (founder) access |
| Data minimization | Only essential data collected |
| Purpose limitation | Data used only for stated purposes |
| Vendor management | DPAs required for all processors |
| Incident response | Breach response plan documented |

---

## Data Subject Rights Procedures

| Right | Process | Response Time |
|-------|---------|---------------|
| **Access** | Email request → Manual database export → Email response | 30 days |
| **Rectification** | Email request → Manual database update → Confirmation | 30 days |
| **Erasure** | Email request → Delete/anonymize records → Confirmation | 30 days |
| **Portability** | Email request → JSON export → Email delivery | 30 days |
| **Objection** | Email request → Case review → Response | 30 days |
| **Restriction** | Email request → Flag records → Confirmation | 30 days |

**Contact for all requests:** hello@chainsights.one

---

## Review Schedule

| Review Type | Frequency | Next Due |
|-------------|-----------|----------|
| ROPA update | Quarterly | 2025-03-16 |
| Processor review | Annually | 2026-12-16 |
| Security review | Annually | 2026-12-16 |
| Policy review | Annually | 2026-12-16 |

---

## Document Control

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2025-12-16 | Daniel Semper (BMAD) | Initial creation |

---

## Disclaimer

This Record of Processing Activities is provided as a compliance template. It should be reviewed by qualified legal counsel and updated as processing activities change. This document does not constitute legal advice.

---

*Generated by Daniel Semper (BMAD Legal Agent) | 2025-12-16*
