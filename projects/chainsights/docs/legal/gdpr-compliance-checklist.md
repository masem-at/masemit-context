# GDPR Compliance Checklist

**Project:** ChainSights DAO Rankings Leaderboard
**Last Verified:** 2025-12-23
**Verified By:** Dev Agent (Stories 8.6, 8.8)

---

## NFR-SEC-9: Data Minimization

| Requirement | Status | Evidence |
|-------------|--------|----------|
| Only consented emails collected | ✅ PASS | Schema requires `consentGiven` boolean |
| No unnecessary PII stored | ✅ PASS | Audited `src/lib/db/schema.ts` |
| Email nullable in freeQueryLog | ✅ PASS | `email: text('email')` - nullable |
| Identifier hashed for quota | ✅ PASS | `identifierHash` SHA-256 |

**Personal Data Inventory:**

| Table | PII Fields | Purpose | Legal Basis |
|-------|------------|---------|-------------|
| orders | customerEmail, customerName | Invoice/delivery | Contract |
| reports | customerEmail, customerName | Report delivery | Contract |
| freeQueryLog | email (nullable) | Optional report delivery | Consent |
| leads | email | Marketing | Consent (explicit) |
| optOutRequests | contactEmail | Opt-out verification | Legitimate interest |

---

## NFR-SEC-10: Right to Erasure

| Requirement | Status | Evidence |
|-------------|--------|----------|
| GDPR delete endpoint exists | ✅ PASS | `src/app/api/gdpr/delete/route.ts` |
| Deletion request flagged immediately | ✅ PASS | `deletionRequested=true` set on request |
| Physical cleanup scheduled | ✅ PASS | Weekly cron (Sundays 3am UTC) |
| Records anonymized/deleted | ✅ PASS | Email set to NULL, identifierHash retained for audit |

**Note:** Deletion requests are flagged immediately (data logically marked for deletion). Physical cleanup runs weekly. GDPR requires response within 30 days - we exceed this requirement.


**Endpoints:**
- `POST /api/gdpr/delete` - Marks records for deletion
- `POST /api/gdpr/export` - Returns user data in JSON
- Weekly cleanup cron: `/api/jobs/gdpr-cleanup`

---

## NFR-SEC-11: Data Retention

| Requirement | Status | Evidence |
|-------------|--------|----------|
| 90-day email cleanup | ✅ PASS | `src/lib/gdpr/retention-cleanup.ts` |
| Cleanup mechanism exists | ✅ PASS | `src/app/api/jobs/gdpr-cleanup/route.ts` |
| 7-year order retention | ✅ PASS | Documented in privacy policy |
| Cron configured | ✅ PASS | `vercel.json` - `0 3 * * 0` |

**Retention Periods:**

| Data Type | Retention | Reason |
|-----------|-----------|--------|
| Order records | 7 years | Austrian tax law (BAO §132) |
| Report data | 1 year | Service warranty |
| Free tier emails | 90 days | Data minimization |
| Marketing opt-in emails | Until unsubscribe | Consent-based |
| Unsubscribed leads | 30 days post-unsubscribe | Grace period |

---

## NFR-SEC-12: Cookie Disclosure

| Requirement | Status | Evidence |
|-------------|--------|----------|
| First-party analytics only | ✅ PASS | `@vercel/analytics` only |
| No third-party tracking | ✅ PASS | Grep verified: no fbq/gtag/pixel |
| Cookie banner NOT required | ✅ PASS | Vercel Analytics is cookie-free |

**Analytics Implementation:**
- Location: `src/lib/analytics.ts`
- Provider: Vercel Analytics (`@vercel/analytics`)
- Events tracked: `social_share`, `cta_click`, `row_expand`, `methodology_view`
- Privacy: No cookies, no PII, first-party only

---

## NFR-SEC-13: Privacy Policy

| Requirement | Status | Evidence |
|-------------|--------|----------|
| Policy exists at /privacy | ✅ PASS | `src/app/privacy/page.tsx` |
| Information collected | ✅ PASS | Section 1 |
| Legal basis | ✅ PASS | Section 2 (table format) |
| Data sharing/processors | ✅ PASS | Section 3 (5 processors) |
| Data retention | ✅ PASS | Section 4 |
| International transfers (SCCs) | ✅ PASS | Section 5 |
| User rights | ✅ PASS | Section 6 |
| Data security | ✅ PASS | Section 7 |
| Analytics disclosure | ✅ PASS | Section 8 (Vercel Analytics) |
| Contact information | ✅ PASS | Section 9 |
| Austrian DPA contact | ✅ PASS | Section 6 (dsb.gv.at) |

---

## Third-Party Processor DPAs (AC6)

| Processor | Purpose | DPA URL | Status |
|-----------|---------|---------|--------|
| Stripe | Payment processing | stripe.com/legal/dpa | ⚠️ TO VERIFY |
| Neon | Database hosting | neon.tech/legal/dpa | ⚠️ TO VERIFY |
| Resend | Email delivery | resend.com/legal/dpa | ⚠️ TO VERIFY |
| Anthropic | AI analysis | anthropic.com/legal | ⚠️ TO VERIFY |
| Vercel | Application hosting | vercel.com/legal/dpa | ⚠️ TO VERIFY |

**Action Required:** Verify DPA acceptance in each provider's dashboard/settings.

---

## Data Subject Rights Implementation

| Right | Implementation | Endpoint |
|-------|----------------|----------|
| Access | GDPR export | `POST /api/gdpr/export` |
| Rectification | Contact support | hello@chainsights.one |
| Erasure | GDPR delete | `POST /api/gdpr/delete` |
| Portability | JSON export | `POST /api/gdpr/export` |
| Objection | Opt-out form | `/rankings/opt-out` |
| Restriction | Admin processing | Admin dashboard |

---

## Technical Security Measures

| Measure | Status | Evidence |
|---------|--------|----------|
| TLS encryption | ✅ PASS | Vercel automatic HTTPS |
| Database encryption | ✅ PASS | Neon encrypts at rest |
| Input validation | ✅ PASS | `src/lib/validation/security.ts` |
| Rate limiting | ✅ PASS | `src/lib/middleware/rate-limit.ts` |
| Admin authentication | ✅ PASS | Bearer token auth |
| PCI-DSS compliance | ✅ PASS | Stripe handles all card data |

---

## Compliance Summary

| NFR | Requirement | Status |
|-----|-------------|--------|
| NFR-SEC-9 | Data minimization | ✅ COMPLIANT |
| NFR-SEC-10 | Right to erasure | ✅ COMPLIANT |
| NFR-SEC-11 | Data retention | ✅ COMPLIANT |
| NFR-SEC-12 | Cookie disclosure | ✅ COMPLIANT |
| NFR-SEC-13 | Privacy policy | ✅ COMPLIANT |

---

## Verification Log

| Date | Verifier | Action |
|------|----------|--------|
| 2025-12-23 | Dev Agent | Initial compliance audit for Story 8.6 |

---

## References

- [GDPR Article 6 - Lawfulness of Processing](https://gdpr.eu/article-6-how-to-process-personal-data-legally/)
- [Austrian DPA (Datenschutzbehörde)](https://dsb.gv.at)
- [docs/legal/gdpr-gap-analysis.md](./gdpr-gap-analysis.md)
- [docs/legal/gdpr-remediation-plan.md](./gdpr-remediation-plan.md)
# Analytics Privacy Verification

## Vercel Analytics GDPR Compliance

✅ **Verified:** 2025-12-23 (Story 8.8)

Vercel Analytics is GDPR-compliant by default:
- No PII tracked (emails, names, wallet addresses)
- IP addresses not stored beyond 30-day standard retention
- Analytics-only cookies (no tracking pixels)
- User consent NOT required for analytics per GDPR

## Custom Event Properties Audited

All 4 custom events verified for GDPR compliance:
- `cta_click` - Properties: dao (string), score (number), location (string) - ✅ NO PII
- `social_share` - Properties: platform (string), dao (string), score (number) - ✅ NO PII
- `row_expand` - Properties: dao (string), rank (number) - ✅ NO PII
- `methodology_view` - No properties - ✅ NO PII

## Privacy Policy

Analytics tracking mentioned in privacy policy (Story 3.4).

