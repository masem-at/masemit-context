# Security Architecture

**Document Version:** 1.0
**Created:** 2025-11-19
**Purpose:** Security patterns, decisions, and implementation strategy for tellingCube

---

## Overview

This document defines the security architecture for tellingCube, including authentication, authorization, rate limiting, data protection, and compliance requirements.

---

## Security Principles

1. **Defense in Depth** - Multiple layers of security controls
2. **Least Privilege** - Minimal access rights for all components
3. **Secure by Default** - Security built in, not bolted on
4. **Fail Secure** - System fails to secure state, not open state
5. **Privacy by Design** - User privacy considered from the start

---

## Rate Limiting Architecture

### Overview

Rate limiting protects against abuse, DoS attacks, and cost overruns by restricting request frequency per client.

### Implementation Strategy

**Technology:** Upstash Redis + @upstash/ratelimit
**Implementation Timeline:** Story 3.9 (Error Handling & Production Polish)
**Pattern:** Sliding window algorithm

### Endpoint Rate Limits

| Endpoint | Limit | Window | Rationale |
|----------|-------|--------|-----------|
| `/api/generate` | 10 requests | 1 hour | High-cost AI operations ($0.15-0.30 per request) |
| `/api/stripe/create-checkout` | 20 requests | 1 hour | Payment operations (prevent session spam) |
| `/api/stripe/webhook` | 100 requests | 1 hour | External webhooks (Stripe may retry) |
| `/api/views/*` | 100 requests | 1 hour | Low-cost data queries |
| `/api/validate` | 50 requests | 1 hour | Validation operations |

### Rate Limiting Middleware Pattern

```typescript
// lib/middleware/rate-limit.ts
import { Ratelimit } from '@upstash/ratelimit'
import { Redis } from '@upstash/redis'

const redis = new Redis({
  url: process.env.UPSTASH_REDIS_REST_URL!,
  token: process.env.UPSTASH_REDIS_REST_TOKEN!,
})

export const RateLimits = {
  generation: { requests: 10, window: '1 h' },
  checkout: { requests: 20, window: '1 h' },
  webhook: { requests: 100, window: '1 h' },
  query: { requests: 100, window: '1 h' },
  validation: { requests: 50, window: '1 h' },
}

export function createRateLimiter(name: keyof typeof RateLimits) {
  const config = RateLimits[name]
  return new Ratelimit({
    redis,
    limiter: Ratelimit.slidingWindow(config.requests, config.window),
    prefix: `ratelimit:${name}`,
  })
}

export async function checkRateLimit(
  request: Request,
  limiter: Ratelimit
): Promise<{ allowed: boolean; limit: number; remaining: number; reset: Date }> {
  const ip =
    request.headers.get('x-forwarded-for')?.split(',')[0] ||
    request.headers.get('x-real-ip') ||
    'unknown'

  const result = await limiter.limit(ip)

  return {
    allowed: result.success,
    limit: result.limit,
    remaining: result.remaining,
    reset: new Date(result.reset),
  }
}
```

### Usage Example

```typescript
// app/api/stripe/create-checkout/route.ts
import { createRateLimiter, checkRateLimit } from '@/lib/middleware/rate-limit'

const checkoutLimiter = createRateLimiter('checkout')

export async function POST(request: Request) {
  // Check rate limit first
  const rateLimit = await checkRateLimit(request, checkoutLimiter)

  if (!rateLimit.allowed) {
    return Response.json(
      {
        error: 'Rate limit exceeded',
        retryAfter: rateLimit.reset.toISOString(),
      },
      {
        status: 429,
        headers: { 'Retry-After': Math.ceil((rateLimit.reset.getTime() - Date.now()) / 1000).toString() }
      }
    )
  }

  // Continue with checkout logic...
}
```

---

## Architectural Decision Record: Rate Limiting Deferral

### Decision

**Story 3.4 (Stripe Checkout Integration)** identified missing rate limiting on `/api/stripe/create-checkout` during QA review. The architectural decision is to **defer implementation to Story 3.9** rather than implement immediately.

### Context

- Story 3.4 creates Stripe checkout API endpoint (backend only)
- Story 3.9 already plans rate limiting infrastructure (Upstash Redis)
- Current deployment is test mode only (no production risk)
- No frontend UI exists yet to call the endpoint (Story 3.5)

### Decision Rationale

1. **Infrastructure Consistency** - Reuse Upstash Redis setup from Story 3.9
2. **Comprehensive Approach** - Address all endpoints requiring rate limiting at once
3. **Avoid Duplicate Work** - Don't set up infrastructure twice
4. **Risk Management** - Current risk is LOW (test mode, no UI, no production deployment)
5. **Cost Optimization** - Single implementation effort vs. multiple iterations

### Consequences

**Positive:**
- Consistent rate limiting pattern across all endpoints
- Single infrastructure setup and configuration
- Comprehensive security review in Story 3.9
- No duplicate work or technical debt

**Negative:**
- Temporary security gap (mitigated by test mode + no production deployment)
- Must ensure Story 3.9 includes all payment endpoints

**Mitigation:**
- Quality gate: No production deployment without rate limiting
- Story 3.9 scope expanded to include payment endpoints
- Security checklist created for Story 3.9 validation

### Status

**Approved:** 2025-11-19
**Approver:** Winston (Architect), Quinn (QA)
**Implementation:** Story 3.9
**Gate:** No production deployment until rate limiting implemented

---

## Payment Security (Stripe)

### PCI Compliance Strategy

**Approach:** Delegate to Stripe Checkout (hosted payment pages)

**Security Model:**
- No credit card data touches our servers
- Stripe handles all payment UI and processing
- We only receive session_id after successful payment
- PCI compliance fully delegated to Stripe

### Stripe API Security

**Server-Side Only:**
- All Stripe API calls from Next.js API routes (server-side)
- `STRIPE_SECRET_KEY` never exposed to client
- Environment variables in `.env.local` (git-ignored)

**Error Handling:**
- Generic error messages to clients (no stack traces)
- Detailed errors logged server-side only
- No exposure of internal Stripe details

**Rate Limiting:**
- Prevent checkout session spam (Story 3.9)
- Protect against Stripe API quota exhaustion
- 20 requests/hour per IP (generous for legitimate users)

---

## Environment Variable Security

### Storage

- Development: `.env.local` (git-ignored)
- Production: Vercel environment variables (encrypted at rest)

### Validation Strategy

**Implementation:** Story 3.9 (startup validation)

**Required Variables:**
```bash
# Database
DATABASE_URL=postgresql://...

# AI Generation
ANTHROPIC_API_KEY=sk-ant-...

# Payment Processing
STRIPE_SECRET_KEY=sk_test_...
STRIPE_PUBLISHABLE_KEY=pk_test_...
NEXT_PUBLIC_BASE_URL=https://...
STRIPE_PRICE_SUPPORTER_S=price_...
STRIPE_PRICE_SUPPORTER_M=price_...
STRIPE_PRICE_SUPPORTER_L=price_...
STRIPE_PRICE_TEAM_PLUS=price_...
STRIPE_PRICE_DEPARTMENT_PARTNER=price_...

# Rate Limiting (Post-PoC)
UPSTASH_REDIS_REST_URL=https://...
UPSTASH_REDIS_REST_TOKEN=...

# Email (Post-PoC)
RESEND_API_KEY=re_...

# Monitoring (Post-PoC)
SENTRY_DSN=https://...
```

### Validation Pattern

```typescript
// lib/config/env-validation.ts
const requiredEnvVars = [
  'DATABASE_URL',
  'ANTHROPIC_API_KEY',
  'STRIPE_SECRET_KEY',
  'STRIPE_PUBLISHABLE_KEY',
  'NEXT_PUBLIC_BASE_URL',
  'STRIPE_PRICE_SUPPORTER_S',
  'STRIPE_PRICE_SUPPORTER_M',
  'STRIPE_PRICE_SUPPORTER_L',
  'STRIPE_PRICE_TEAM_PLUS',
  'STRIPE_PRICE_DEPARTMENT_PARTNER',
]

export function validateEnvironment() {
  const missing = requiredEnvVars.filter(key => !process.env[key])

  if (missing.length > 0) {
    throw new Error(
      `Missing required environment variables:\n${missing.join('\n')}\n\n` +
      `Please check your .env.local file and ensure all required variables are set.`
    )
  }
}

// Call at app startup (before any requests)
validateEnvironment()
```

---

## Authentication & Authorization (Post-PoC)

**MVP Scope:** No authentication (ephemeral sessions only)

**Post-PoC Strategy:**
- Email-based authentication (magic links)
- Session management with secure cookies
- Founding member status tied to email/payment
- JWT tokens for API access

**Future Consideration:** OAuth providers (Google, GitHub) for enterprise users

---

## Data Protection

### Database Security

**Technology:** NeonDB (PostgreSQL) with SSL/TLS

**Access Control:**
- Database credentials in environment variables only
- Prisma connection string uses SSL mode
- No direct database access from frontend

**Data Encryption:**
- In transit: TLS 1.3 for all connections
- At rest: NeonDB encryption (managed by provider)

### Sensitive Data Handling

**Payment Data:**
- Never stored in our database
- Stripe session_id only (used for verification)
- Payment confirmations via Stripe webhooks

**User Data (Post-PoC):**
- Email addresses hashed for lookup
- No personally identifiable information in logs
- GDPR compliance for EU users

---

## API Security

### Input Validation

**Strategy:** Validate all inputs at API boundary

**Pattern:**
```typescript
// Validate tier parameter
if (!tier || typeof tier !== 'string') {
  return Response.json({ error: 'Invalid input' }, { status: 400 })
}

// Validate against known values
if (!ALLOWED_TIERS.includes(tier)) {
  return Response.json({ error: 'Invalid tier' }, { status: 400 })
}
```

### Error Handling

**Client Responses:**
- Generic error messages only
- No stack traces or internal details
- Standard HTTP status codes

**Server Logging:**
- Detailed errors logged for debugging
- Include request context (IP, timestamp, endpoint)
- Integration with Sentry (Story 3.9)

### CORS Policy

**Current:** Same-origin only (Next.js default)

**Future:** Whitelist specific domains for white-label API access

---

## Security Monitoring (Story 3.9)

### Error Tracking

**Tool:** Sentry
**Coverage:** API failures, validation errors, payment webhook issues

### Metrics to Monitor

- Rate limit hit rate (how often users hit limits)
- Failed payment attempts
- Validation failures (potential malicious input)
- AI generation errors

### Alerting Strategy

- Immediate: Payment processing failures
- Daily: Rate limit patterns, error rates
- Weekly: Security review summary

---

## Security Checklist for Story 3.9

### Rate Limiting Implementation

- [ ] Install Upstash dependencies (`@upstash/ratelimit`, `@upstash/redis`)
- [ ] Configure Upstash Redis (environment variables)
- [ ] Create rate limiting middleware (`lib/middleware/rate-limit.ts`)
- [ ] Apply rate limiting to all sensitive endpoints:
  - [ ] `/api/generate` (AI generation)
  - [ ] `/api/stripe/create-checkout` (payment)
  - [ ] `/api/stripe/webhook` (if exists)
  - [ ] Review other endpoints for rate limit needs
- [ ] Test rate limiting with automated tests
- [ ] Verify 429 responses include `Retry-After` headers
- [ ] Document rate limits in API documentation

### Environment Validation

- [ ] Create startup validation function
- [ ] Validate all required environment variables
- [ ] Provide helpful error messages for missing vars
- [ ] Test startup with missing variables (should fail gracefully)
- [ ] Document all required environment variables

### Error Monitoring

- [ ] Set up Sentry account and project
- [ ] Install Sentry SDK
- [ ] Configure Sentry DSN in environment variables
- [ ] Integrate Sentry with API error handling
- [ ] Test error reporting
- [ ] Set up alerting rules

### Security Review

- [ ] Review all API endpoints for input validation
- [ ] Check all error responses (no info leakage)
- [ ] Verify no secrets in client-side code
- [ ] Test rate limiting under load
- [ ] Review CORS policy
- [ ] Document security decisions

---

## Production Deployment Gate

### Security Requirements (Must Pass Before Production)

1. ✅ **Rate limiting implemented** on all sensitive endpoints
2. ✅ **Environment validation** at startup (fail-fast)
3. ✅ **Error monitoring** (Sentry) configured and tested
4. ✅ **Input validation** on all API endpoints
5. ✅ **No secrets exposed** to client-side code
6. ✅ **HTTPS enforced** (via Vercel)
7. ✅ **Database connections** use SSL/TLS
8. ✅ **Generic error messages** to clients (no info leakage)

**Gate Owner:** Quinn (QA)
**Enforced By:** Quality gate review before production deployment

---

## Threat Model (Future Work)

### Identified Threats

1. **API Abuse** - Mitigated by rate limiting
2. **DoS Attacks** - Mitigated by rate limiting + Vercel DDoS protection
3. **Payment Fraud** - Mitigated by Stripe's fraud detection
4. **Data Breach** - Mitigated by minimal data storage + encryption
5. **Injection Attacks** - Mitigated by Prisma (parameterized queries) + input validation

### Future Threat Modeling

- [ ] Formal threat modeling workshop (post-MVP)
- [ ] Penetration testing (Alpha phase)
- [ ] Security audit (before public launch)

---

## Change Log

| Date | Version | Change | Author |
|------|---------|--------|--------|
| 2025-11-19 | 1.0 | Initial security architecture document | Winston (Architect) |

---

## References

- [Story 3.4 QA Review](../stories/3.4-stripe-checkout-integration.md#qa-results)
- [Story 3.9 Specification](../stories/3.9-error-handling-production-polish.md)
- [Tech Stack](./tech-stack.md)
- [Upstash Rate Limiting Documentation](https://upstash.com/docs/redis/features/ratelimiting)
- [Stripe Security Best Practices](https://stripe.com/docs/security/guide)
