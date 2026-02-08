# MMS Party & Consent Integration Guide

**Status:** Production (ChainSights reference implementation)
**Owner:** masemIT
**Audience:** Developers integrating MMS Party & Consent APIs into masemIT projects (TellingCube, etc.)
**API Docs:** https://api.masem.at/docs

---

## 1. Overview

The MMS Party & Consent APIs provide centralized lead management and GDPR-compliant consent tracking across all masemIT projects. Instead of each project managing its own contact lists and consent records, MMS acts as the single source of truth.

**What it gives you:**
- Find-or-create contact management (no duplicates across projects)
- Party lifecycle tracking (lead → prospect → customer → churned)
- GDPR consent recording (immutable audit trail)
- Cookie consent tracking via device IDs
- Marketing email preferences with self-service unsubscribe (public endpoints, no auth needed)
- Cross-project visibility (a customer in ChainSights is visible to TellingCube if needed)

---

## 2. Prerequisites

### API Key

Request an API key from Mario. Format: `mms_{project}_{secret}`

### Required Scopes

Your API key **must** have these scopes:

```
parties:read
parties:write
consents:read
consents:write
```

Without these scopes, the API will return `403 FORBIDDEN`.

### Environment Variables

```env
MMS_API_URL=https://api.masem.at
MMS_API_KEY=mms_yourproject_xxxxx
```

### Source Project Enum

Your project must be registered as a `source_project` in MMS. Current values:

```
hoki_help | tellingcube | chainsights | masem_at | manual
```

If your project is not listed, ask Mario to add it.

---

## 3. Architecture Overview

```
User Action (sign up, purchase, free tool)
        │
        ▼
Your Backend ──POST /v1/parties──► MMS (find-or-create)
        │                              │
        │                              ├─ Party record (lead)
        │                              ├─ Contact point (email)
        │                              ├─ Consent record (if marketing_consent)
        │                              └─ Email preference + unsubscribe_token
        │
        ▼
Email with unsubscribe link ──► /unsubscribe?token=xxx
                                        │
                                        ▼
                                POST /v1/email-preferences/unsubscribe (PUBLIC, no auth)
```

**Cookie Consent Flow:**

```
First Visit ──► Cookie Banner (Accept/Decline)
                    │
                    ├─ localStorage: cookie_consent = granted|denied
                    ├─ localStorage: cookie_device_id = uuid
                    │
                    ▼
Your Backend ──POST /v1/consents──► MMS (audit trail)
                    │
                    ▼
            Tracker loads only if granted
```

---

## 4. Implementation Patterns

### 4.1 Party Creation (Lead Capture)

The core pattern: whenever you collect an email, create a party in MMS.

```typescript
// lib/mms.ts

interface CreatePartyOptions {
  marketingConsent?: boolean
  consentIp?: string
  consentUserAgent?: string
}

interface CreatePartyResult {
  partyId: string
  contactPointId: string
  created: boolean
  unsubscribeToken: string | null  // only when marketingConsent=true AND new party
}

export async function createParty(
  email: string,
  options?: CreatePartyOptions
): Promise<CreatePartyResult | null> {
  const baseUrl = process.env.MMS_API_URL
  const apiKey = process.env.MMS_API_KEY
  if (!baseUrl || !apiKey) return null

  const body: Record<string, unknown> = {
    email,
    display_name: email,
    party_type: 'person',
    source_project: 'yourproject',  // ← change this
  }

  if (options?.marketingConsent !== undefined) {
    body.marketing_consent = options.marketingConsent
  }
  if (options?.consentIp) body.consent_ip = options.consentIp
  if (options?.consentUserAgent) body.consent_user_agent = options.consentUserAgent
  if (options?.marketingConsent) body.consent_text_version = 'yourproject-v1'

  const response = await fetch(`${baseUrl}/v1/parties`, {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'x-api-key': apiKey,
    },
    body: JSON.stringify(body),
  })

  if (!response.ok) return null

  const data = await response.json()
  const result = data.data || data
  return {
    partyId: result.party_id || result.id,
    contactPointId: result.contact_point_id || '',
    created: result.created ?? true,
    unsubscribeToken: result.unsubscribe_token || null,
  }
}
```

**Key behaviors:**
- Find-or-create: if email exists, returns existing party (`created: false`)
- `unsubscribe_token` only returned when `marketing_consent: true` AND party is newly created
- If party already exists with marketing consent, token is `null` (already issued)

### 4.2 Party Status Upgrade

After a purchase, upgrade the party from `lead` to `customer`:

```typescript
export async function updatePartyStatus(
  partyId: string,
  status: 'lead' | 'prospect' | 'customer' | 'key_account' | 'churned'
): Promise<void> {
  const baseUrl = process.env.MMS_API_URL
  const apiKey = process.env.MMS_API_KEY
  if (!baseUrl || !apiKey) return

  await fetch(`${baseUrl}/v1/parties/${partyId}`, {
    method: 'PATCH',
    headers: {
      'Content-Type': 'application/json',
      'x-api-key': apiKey,
    },
    body: JSON.stringify({ party_status: status }),
  }).catch(() => {})  // fire-and-forget
}
```

### 4.3 Payment Webhook Pattern

In your Stripe (or other payment) webhook, create the party and upgrade:

```typescript
// Fire-and-forget — must never block/fail the webhook
createParty(customerEmail).then(async (result) => {
  if (result?.partyId) {
    await updatePartyStatus(result.partyId, 'customer')
  }
}).catch((err) => console.error('[Webhook] MMS createParty failed:', err))
```

**Important:** Payment webhooks have timeouts (Stripe: 10s). MMS calls must be non-blocking.

### 4.4 Unsubscribe Flow

Three pieces needed:

**A) Include unsubscribe link in marketing emails:**
```
https://yourproject.com/unsubscribe?token={unsubscribe_token}
```

Plus email headers:
```
List-Unsubscribe: <https://yourproject.com/unsubscribe?token=xxx>
List-Unsubscribe-Post: List-Unsubscribe=One-Click
```

**B) API proxy route** (to avoid exposing MMS API structure to the client):
```typescript
// POST /api/unsubscribe  — your backend
const response = await fetch(`${MMS_API_URL}/v1/email-preferences/unsubscribe`, {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    token: requestBody.token,
    preference_type: 'newsletter',
  }),
})
```

Note: The MMS unsubscribe endpoint is **public** (no API key needed), but proxying hides the MMS URL and adds your own error handling.

**C) Unsubscribe page** at `/unsubscribe?token=xxx`:
- Read token from URL
- Show "Unsubscribe" button
- Call your `/api/unsubscribe` proxy
- Show confirmation

### 4.5 Cookie Consent Banner

**Client-side flow:**

1. Generate `device_id` via `crypto.randomUUID()`, store in `localStorage`
2. Show banner with **equal** Accept/Decline buttons (GDPR requirement)
3. On choice: `localStorage.setItem('cookie_consent', 'granted' | 'denied')`
4. Record to MMS via your backend proxy:

```typescript
// POST /api/consent  — your backend proxies to MMS
await fetch(`${MMS_API_URL}/v1/consents`, {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'x-api-key': apiKey,
  },
  body: JSON.stringify({
    device_id: body.device_id,
    consent_type: 'cookie_analytics',
    scope: 'project',
    granted: body.granted,
    ip_address: clientIp,
    user_agent: clientUserAgent,
    consent_text_version: 'yourproject-cookie-banner-v1',
  }),
})
```

**Tracker gating:**

```typescript
// Only load tracker when consent is granted
if (localStorage.getItem('cookie_consent') === 'granted') {
  // load analytics script
}
```

**Important:** The consent API proxy should be **non-blocking** — localStorage is the primary gate, MMS is the audit trail. If MMS is down, the banner still works.

### 4.6 Consent Check (optional)

To verify consent state server-side (e.g., for SSR):

```
GET /v1/consents/check?device_id=xxx&consent_type=cookie_analytics
```

Returns `{ granted: true/false, last_updated: "..." }` or `404` if no consent recorded.

---

## 5. Edge Cases & Gotchas

| Scenario | Behavior |
|----------|----------|
| Same email across projects | MMS find-or-create returns the same party. `source_project` tracks where they first appeared. Both projects can see the party. |
| User gets free tool → then purchases | `createParty` returns existing lead, webhook PATCHes to `customer`. Marketing consent preserved. |
| User purchases first → then uses free tool with marketing opt-in | `createParty` finds existing customer, adds marketing consent, generates `unsubscribe_token`. |
| User unsubscribes → new consent via different tool | New consent record created (append-only). Old revocation preserved in audit trail. |
| MMS API down during webhook | Fire-and-forget pattern means webhook still succeeds. Party creation retried on next interaction. |
| `unsubscribe_token` is null | Party already existed (not newly created). Token was issued on first creation. To get the token for an existing party, use `GET /v1/parties/:id` and check email preferences. |
| Consent records are immutable | Never updated or deleted. Each change (grant, revoke) creates a new record. This is GDPR audit trail by design. |

---

## 6. Privacy Policy Template Sections

Copy-paste these into your privacy policy and adapt project-specific details.

### Data Collection Section

> **[Your Feature Name]**
>
> When you use [feature], we collect:
> - Email address (for [purpose])
> - Optional marketing consent (for newsletter — opt-in only)
> - IP address (rate limiting and consent audit trail)
>
> Your email is stored in our lead management system (masemIT Management System, hosted in Austria, EU) for communication and order management. If you opt into marketing emails, an unsubscribe link is included in every email.

### Legal Basis Table Rows

> | [Feature] delivery | Performance of contract (Art. 6(1)(b)) |
> | [Feature] marketing consent | Consent (Art. 6(1)(a)) |
> | Cookie analytics | Consent (Art. 6(1)(a)) |

### Data Processor Table Row

> | masemIT (MMS) | Lead & consent management | Austria (EU) | GDPR |

### Cookie Table Rows

> | [your_session_cookie] | Authentication | [duration] | Necessary (Art. 6(1)(b)) |
> | masemIT tracker | Analytics | Session | Consent (Art. 6(1)(a)) |

### Analytics Section

> **masemIT Analytics (consent required)**
>
> We use a self-hosted analytics tracker (analytics.masem.at) to measure page views and engagement. This tracker uses cookies and is only loaded after you accept analytics cookies via the consent banner. If you decline, no tracking data is collected. The tracker is hosted in Austria (EU) and data is processed under GDPR.

---

## 7. Integration Checklist

Before launch, verify:

- [ ] API key obtained with scopes `parties:read`, `parties:write`, `consents:read`, `consents:write`
- [ ] `MMS_API_URL` and `MMS_API_KEY` in environment variables
- [ ] `source_project` enum value registered in MMS for your project
- [ ] `createParty` called on every email collection touchpoint
- [ ] `marketing_consent`, `consent_ip`, `consent_user_agent` passed when consent checkbox present
- [ ] `unsubscribe_token` stored/used from `createParty` response
- [ ] Unsubscribe link in every marketing email (`List-Unsubscribe` header + footer link)
- [ ] `/unsubscribe` page functional with token from URL
- [ ] Payment webhook calls `createParty` + `updatePartyStatus('customer')` (fire-and-forget)
- [ ] Cookie consent banner with equal Accept/Decline buttons
- [ ] Analytics tracker gated behind `cookie_consent === 'granted'` in localStorage
- [ ] Consent API proxy route forwards to MMS (keeps API key server-side)
- [ ] Privacy policy updated with: data collection, legal basis, MMS as processor, cookies, analytics
- [ ] `consent_text_version` set to a versioned string (e.g., `yourproject-cookie-banner-v1`)

---

## 8. MMS API Quick Reference

| Endpoint | Method | Auth | Purpose |
|----------|--------|------|---------|
| `/v1/parties` | POST | `parties:write` | Find-or-create party by email |
| `/v1/parties/lookup?email=x` | GET | `parties:read` | Lookup party by email |
| `/v1/parties/:id` | GET | `parties:read` | Get party by ID |
| `/v1/parties/:id` | PATCH | `parties:write` | Update party (status, name) |
| `/v1/parties/:id/contact-points` | POST | `parties:write` | Add contact point |
| `/v1/parties/:id/consents` | GET | `consents:read` | List party consents |
| `/v1/consents` | POST | `consents:write` | Record consent (cookie, marketing) |
| `/v1/consents/check` | GET | `consents:read` | Check consent status |
| `/v1/email-preferences/:token` | GET | **Public** | Get email preferences |
| `/v1/email-preferences/unsubscribe` | POST | **Public** | Unsubscribe by token |
| `/v1/email-preferences/:token` | PATCH | **Public** | Update preferences |

Full docs: https://api.masem.at/docs/parties and https://api.masem.at/docs/consents

---

## 9. Lessons Learned (ChainSights)

1. **Fire-and-forget for webhooks.** MMS calls in payment webhooks must never block or throw. Stripe has a 10s timeout, and a failed MMS call should not prevent order creation.

2. **localStorage is the gate, MMS is the audit trail.** The cookie consent banner works entirely via localStorage. The MMS API call is for GDPR compliance records. If MMS is down, the banner still functions correctly.

3. **Consent API proxy is essential.** Never expose `MMS_API_KEY` to the client. Create a thin `/api/consent` proxy that forwards to MMS with the server-side key. Return success even if MMS fails (non-blocking).

4. **`unsubscribe_token` is only returned once.** On party creation with `marketing_consent: true`. If the party already exists, token is `null`. If you need the token later, you need to query email preferences via the party ID.

5. **Privacy policy is not optional content.** GDPR requires specific sections: purpose, legal basis, processors, retention, rights. Use the templates in Section 6 as a starting point — they cover the required fields.

6. **Accept and Decline must be equal.** GDPR/ePrivacy requires that declining cookies is as easy as accepting. No dark patterns (small decline button, pre-checked boxes, etc.).

7. **`consent_text_version` enables audit.** If you ever change your cookie banner text or consent checkbox wording, bump the version string. MMS stores it per consent record so you can prove which text the user agreed to.
