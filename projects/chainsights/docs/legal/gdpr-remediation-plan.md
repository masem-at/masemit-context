# GDPR Remediation Plan: ChainSights

**Date:** 2025-12-16
**Analyst:** Daniel Semper (BMAD Legal)
**Status:** Draft - Requires Legal Review

---

## Remediation Overview

This plan addresses the gaps identified in the GDPR Gap Analysis, prioritized by risk and effort.

**Total Estimated Effort:** 3-5 days
**Priority:** Complete before public marketing launch

---

## Immediate Actions (Do This Week)

### 1. Sign Data Processing Agreements

**Effort:** 1-2 hours
**Risk Reduction:** High

Accept DPAs from all your processors. Most have online acceptance:

| Processor | DPA Link | Action |
|-----------|----------|--------|
| Stripe | [stripe.com/legal/dpa](https://stripe.com/legal/dpa) | Accept in Dashboard → Settings → Compliance |
| Vercel | [vercel.com/legal/dpa](https://vercel.com/legal/dpa) | Accept in account settings |
| Neon | [neon.tech/legal/dpa](https://neon.tech/legal/dpa) | Email acceptance or dashboard |
| Resend | [resend.com/legal/dpa](https://resend.com/legal/dpa) | Accept in dashboard |
| Anthropic | [anthropic.com/legal](https://www.anthropic.com/legal/commercial-terms) | Review commercial terms (DPA included) |
| Upstash | [upstash.com/trust/dpa](https://upstash.com/trust/dpa) | Accept in dashboard |

**Checklist:**
- [ ] Stripe DPA accepted
- [ ] Vercel DPA accepted
- [ ] Neon DPA accepted
- [ ] Resend DPA accepted
- [ ] Anthropic terms reviewed
- [ ] Upstash DPA accepted
- [ ] Screenshot/save confirmation of each

---

### 2. Update Privacy Policy

**Effort:** 2-3 hours
**Risk Reduction:** High

Update `/privacy` page with the following additions:

#### Add: Complete Data Recipients List

```markdown
## Data Sharing

We share your data with the following service providers:

| Provider | Purpose | Location | Safeguards |
|----------|---------|----------|------------|
| Stripe | Payment processing | USA | Standard Contractual Clauses |
| Neon | Database hosting | Germany (Frankfurt) | EU/GDPR Compliant |
| Resend | Email delivery | Ireland | EU/GDPR Compliant |
| Anthropic | AI analysis | USA | Standard Contractual Clauses |
| Vercel | Application hosting | Germany (Frankfurt) | EU/GDPR Compliant |

All providers have signed Data Processing Agreements and comply with GDPR requirements.
```

#### Add: Data Retention Section

```markdown
## Data Retention

We retain your data for the following periods:

| Data Type | Retention Period | Reason |
|-----------|------------------|--------|
| Order records | 7 years | Austrian tax/accounting requirements |
| Report data | 1 year after delivery | Service warranty and support |
| Email address | Until deletion requested | Service delivery and communication |

After these periods, data is automatically deleted unless required for legal obligations.
```

#### Add: Your Rights Section

```markdown
## Your Rights

Under GDPR, you have the following rights:

- **Access:** Request a copy of your personal data
- **Rectification:** Correct inaccurate data
- **Erasure:** Request deletion of your data ("right to be forgotten")
- **Portability:** Receive your data in a machine-readable format
- **Objection:** Object to processing based on legitimate interests
- **Restriction:** Request limited processing in certain circumstances

To exercise any of these rights, contact us at hello@chainsights.one. We will respond within 30 days.

If you believe your rights have been violated, you may lodge a complaint with the Austrian Data Protection Authority (Datenschutzbehörde) at dsb.gv.at.
```

#### Add: International Transfers Section

```markdown
## International Data Transfers

Your data may be transferred to service providers in the United States. These transfers are protected by Standard Contractual Clauses (SCCs) approved by the European Commission, ensuring your data receives equivalent protection outside the EU.
```

#### Add: Legal Basis Section

```markdown
## Legal Basis for Processing

| Processing Activity | Legal Basis |
|---------------------|-------------|
| Report generation | Performance of contract (Art. 6(1)(b)) |
| Payment processing | Performance of contract (Art. 6(1)(b)) |
| Email communication | Performance of contract (Art. 6(1)(b)) |
| Service improvement | Legitimate interest (Art. 6(1)(f)) |
```

---

## Short-Term Actions (Next 30 Days)

### 3. Implement Data Subject Rights Endpoints

**Effort:** 1-2 days development
**Risk Reduction:** High

Create endpoints for data access and deletion:

#### Data Export Endpoint

```typescript
// POST /api/user/export
// Request: { email: string }
// Response: JSON with all user data

// Implementation:
// 1. Verify email ownership (send confirmation code)
// 2. Query orders and reports by email
// 3. Return JSON package with all data
// 4. Log the request for audit trail
```

#### Data Deletion Endpoint

```typescript
// POST /api/user/delete
// Request: { email: string }
// Response: { success: boolean, deletedRecords: number }

// Implementation:
// 1. Verify email ownership (send confirmation code)
// 2. Delete or anonymize records in orders/reports tables
// 3. Retain anonymized record for accounting (7 years)
// 4. Log the deletion request for audit trail
// 5. Send confirmation email
```

#### Manual Process (Interim)

Until endpoints are built, document this manual process:

1. User emails hello@chainsights.one with subject "GDPR Request"
2. Verify identity (ask for order confirmation email)
3. Export: Query database, send JSON via email
4. Delete: Run deletion query, confirm via email
5. Log all requests in spreadsheet
6. Respond within 30 days (GDPR requirement)

---

### 4. Define Data Retention Policy

**Effort:** 2-4 hours
**Risk Reduction:** High

#### Retention Schedule

| Table | Field | Retention | Action After |
|-------|-------|-----------|--------------|
| orders | All fields | 7 years | Archive then delete |
| reports | rawData | 30 days | Set to NULL |
| reports | aiAnalysis | 1 year | Set to NULL |
| reports | finalPdfUrl | 1 year | Set to NULL |
| reports | customerEmail | 1 year | Anonymize |
| share_rewards | All fields | 1 year | Delete |

#### Implementation

Create a scheduled job (QStash or cron):

```typescript
// Run weekly: cleanupOldData()
// 1. Find reports older than 1 year
// 2. Set rawData, aiAnalysis, finalPdfUrl to NULL
// 3. Replace customerEmail with hash
// 4. Delete share_rewards older than 1 year
// 5. Log cleanup actions
```

---

### 5. Create Breach Response Plan

**Effort:** 2-3 hours
**Risk Reduction:** Medium

#### Breach Response Playbook

**Step 1: Detection & Containment (0-4 hours)**
- Identify scope of breach
- Contain the incident (revoke access, patch vulnerability)
- Preserve evidence

**Step 2: Assessment (4-24 hours)**
- Determine data affected
- Assess risk to individuals
- Document timeline

**Step 3: Notification Decision (24-48 hours)**

| Risk Level | DPA Notification | User Notification |
|------------|------------------|-------------------|
| Low (no real risk) | Not required | Not required |
| Medium (some risk) | Required within 72h | Recommended |
| High (likely harm) | Required within 72h | Required without delay |

**Step 4: Notification (if required)**

DPA Contact:
```
Datenschutzbehörde
Barichgasse 40-42
1030 Vienna, Austria
dsb@dsb.gv.at
+43 1 52 152-0
```

User Notification Template:
```
Subject: Security Notification - ChainSights

We are writing to inform you of a security incident affecting your data.

What happened: [Description]
What data was affected: [List]
What we're doing: [Actions taken]
What you can do: [Recommendations]

Contact: hello@chainsights.one
```

**Step 5: Post-Incident**
- Root cause analysis
- Update security measures
- Document lessons learned

---

## Medium-Term Actions (Next 90 Days)

### 6. Implement Security Improvements

**Effort:** 1 day development
**Risk Reduction:** Medium

#### Hash Admin Password

```typescript
// Install bcrypt
// npm install bcrypt @types/bcrypt

// In auth.ts:
import bcrypt from 'bcrypt'

export async function validateAdminPassword(password: string): Promise<boolean> {
  const hashedPassword = process.env.ADMIN_PASSWORD_HASH
  if (!hashedPassword) return false
  return bcrypt.compare(password, hashedPassword)
}

// Generate hash for .env:
// node -e "require('bcrypt').hash('yourpassword', 10).then(console.log)"
```

#### Add Rate Limiting

```typescript
// Add to API routes
// npm install @upstash/ratelimit

import { Ratelimit } from '@upstash/ratelimit'
import { Redis } from '@upstash/redis'

const ratelimit = new Ratelimit({
  redis: Redis.fromEnv(),
  limiter: Ratelimit.slidingWindow(10, '1 m'), // 10 requests per minute
})
```

---

### 7. Create Lightweight DPIA for AI Processing

**Effort:** 2-3 hours
**Risk Reduction:** Low-Medium

Document the following:

```markdown
# Data Protection Impact Assessment: AI Analysis

## Processing Description
ChainSights uses Anthropic Claude to analyze governance data and generate reports.

## Data Processed
- DAO governance metrics (public blockchain data)
- Voter addresses (public blockchain data)
- Voting patterns (derived from public data)

## Risk Assessment
- **Profiling:** No - reports are informational, not used for automated decisions
- **Automated Decision-Making:** No - all reports reviewed by human before delivery
- **Sensitive Data:** No - only public blockchain data processed

## Safeguards
- Data Processing Agreement with Anthropic
- Standard Contractual Clauses for US transfer
- Data minimization (only necessary governance data sent)
- No personal data beyond what's publicly visible on-chain

## Conclusion
Low risk processing. Standard safeguards sufficient.
```

---

### 8. Add Audit Logging

**Effort:** 4-8 hours development
**Risk Reduction:** Low

Create audit log table:

```typescript
// Schema addition
export const auditLogs = pgTable('audit_logs', {
  id: uuid('id').defaultRandom().primaryKey(),
  action: text('action').notNull(), // 'view_report', 'export_data', 'delete_data'
  actorType: text('actor_type').notNull(), // 'admin', 'system', 'user'
  actorId: text('actor_id'),
  targetType: text('target_type'), // 'report', 'order'
  targetId: uuid('target_id'),
  metadata: jsonb('metadata'),
  createdAt: timestamp('created_at').defaultNow().notNull(),
})
```

---

## Ongoing Compliance

### Quarterly Review Checklist

- [ ] Review and update privacy policy if needed
- [ ] Check DPA status with processors
- [ ] Run data retention cleanup
- [ ] Review access logs for anomalies
- [ ] Update processor list if new services added

### Annual Review

- [ ] Full GDPR compliance audit
- [ ] Update DPIA if processing changes
- [ ] Review retention periods
- [ ] Staff training (if applicable)
- [ ] Review and test breach response plan

---

## Resource Needs

### Internal Effort

| Task | Estimated Hours |
|------|-----------------|
| Sign DPAs | 2h |
| Update privacy policy | 3h |
| Data rights endpoints | 8-16h |
| Retention policy + job | 4h |
| Breach response plan | 3h |
| Security improvements | 8h |
| DPIA documentation | 3h |
| **Total** | **31-39h** |

### External Help Needed

| Need | When | Estimated Cost |
|------|------|----------------|
| Legal review of privacy policy | Before launch | €500-1,500 |
| Legal review of terms | Before launch | €500-1,500 |
| Full GDPR audit (if scaling) | Before Series A | €3,000-10,000 |

### Tools to Consider

| Tool | Purpose | Cost |
|------|---------|------|
| OneTrust | Compliance management | Enterprise pricing |
| Privasee | Privacy policy generator | €50-200/month |
| Transcend | Data subject request automation | Usage-based |

**Recommendation:** For MVP phase, manual processes + updated policies are sufficient. Invest in tools when you hit 100+ customers.

---

## Disclaimer

This remediation plan is provided as a template for internal compliance planning. It does not constitute legal advice. Implementation should be reviewed by qualified legal counsel to ensure compliance with Austrian and EU requirements.

---

*Generated by Daniel Semper (BMAD Legal Agent) | 2025-12-16*
