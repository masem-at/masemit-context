# Story: MMS Consent & Party Integration

**Date:** 2026-02-08
**Decision Source:** Party Mode Session (Mary, John, Winston, Paige)
**Status:** PLANNED
**Blocked by:** Nothing — can start immediately
**Must complete before:** Heavy Score Card promotion

---

## Context

Score Card shipped 2026-02-08 with `createParty(email)` for lead capture, but:
- `createParty` does NOT pass `marketing_consent`, `consent_ip`, `consent_user_agent`
- `unsubscribe_token` is never returned or used
- Marketing consent is only logged to LogSnag (not persisted in MMS)
- Paid reports (Deep Dive, Governance Audit) do NOT call `createParty` at all
- masemIT tracker loads unconditionally in `layout.tsx` — no cookie consent
- Privacy Policy missing: Score Card, MMS, masemIT tracker, `cs_session` cookie

**MMS API Docs:** https://api.masem.at/docs

---

## Phase 1: API Layer (prerequisite for everything)

### 1.1 Extend `createParty` in `src/lib/mms/analytics.ts`

**Current signature:**
```ts
createParty(email: string): Promise<{ partyId: string; created: boolean } | null>
```

**New signature:**
```ts
interface CreatePartyOptions {
  marketingConsent?: boolean
  consentIp?: string
  consentUserAgent?: string
}

interface CreatePartyResult {
  partyId: string
  contactPointId: string
  created: boolean
  unsubscribeToken: string | null
}

createParty(email: string, options?: CreatePartyOptions): Promise<CreatePartyResult | null>
```

**Request body change** — pass to `POST /v1/parties`:
```json
{
  "email": "...",
  "display_name": "...",
  "party_type": "person",
  "source_project": "chainsights",
  "marketing_consent": true,
  "consent_ip": "1.2.3.4",
  "consent_user_agent": "Mozilla/...",
  "consent_text_version": "chainsights-scorecard-v1"
}
```

**Response** — extract `unsubscribe_token` (only returned when `marketing_consent: true` and party is newly created).

### 1.2 New helper `updatePartyStatus` in same file

```ts
updatePartyStatus(partyId: string, status: 'lead' | 'customer'): Promise<void>
```

Calls `PATCH /v1/parties/:id` with `{ party_status: status }`. Fire-and-forget, never throws.

---

## Phase 2: Unsubscribe + Paid Report Lead Capture (P1)

### 2.1 Score Card Route — pass consent fields

**File:** `src/app/api/score-card/route.ts`

- Extract `ip` (already available via `getClientIp`) and `user-agent` from request headers
- Pass to `createParty(email, { marketingConsent, consentIp, consentUserAgent })`
- Await result (not fire-and-forget) to get `unsubscribeToken`
- Pass `unsubscribeToken` to `sendScoreCardEmail` (only if marketing consent given)

### 2.2 Score Card Email — add unsubscribe link

**File:** `src/lib/email/index.ts` (function `sendScoreCardEmail`)

- Add optional `unsubscribeToken` parameter
- If present, add footer: "You opted in to marketing emails. [Unsubscribe](https://chainsights.one/unsubscribe?token=xxx)"
- Add `List-Unsubscribe` header for email clients

### 2.3 Unsubscribe Page

**New file:** `src/app/unsubscribe/page.tsx`

- Server Component
- Read `token` from searchParams
- If no token → show error
- Show "Unsubscribe" button → calls client action
- Client action: `POST https://api.masem.at/v1/email-preferences/unsubscribe` with `{ token, preference_type: "newsletter" }`
  - Note: This is a PUBLIC endpoint (no auth needed), but must be called server-side to avoid CORS
  - So: create `src/app/api/unsubscribe/route.ts` as proxy, or use Server Action
- Show confirmation: "You have been unsubscribed from marketing emails."

### 2.4 Stripe Webhook — createParty + status upgrade

**File:** `src/app/api/webhooks/stripe/route.ts` (function `handleCheckoutCompleted`)

After order insert (line ~186), add fire-and-forget:

```ts
createParty(customerEmail).then(async (result) => {
  if (result?.partyId) {
    await updatePartyStatus(result.partyId, 'customer')
  }
}).catch((err) => console.error('[Webhook] MMS createParty failed:', err))
```

- No `marketing_consent` — paid reports are transactional (Art. 6(1)(b))
- Status `customer` distinguishes paying users from free leads
- Must not block or fail the webhook (Stripe 10s timeout)

---

## Phase 3: Cookie Consent Banner (P0-A)

### 3.1 CookieConsentBanner Component

**New file:** `src/components/CookieConsentBanner.tsx` (Client Component)

**Behavior:**
1. On mount: check `localStorage.getItem('cookie_consent')`
2. If `'granted'` or `'denied'` → don't show banner
3. If not set → show banner with two equal buttons: "Accept" / "Decline"
4. On Accept:
   - `localStorage.setItem('cookie_consent', 'granted')`
   - `localStorage.setItem('device_id', crypto.randomUUID())` (if not already set)
   - Call `POST /v1/consents` via internal API proxy with: `{ device_id, consent_type: "cookie_analytics", granted: true, ip_address, user_agent }`
   - Trigger tracker load (event or state)
5. On Decline:
   - `localStorage.setItem('cookie_consent', 'denied')`
   - Same API call with `granted: false`
   - Tracker does NOT load

**API proxy** (to avoid exposing MMS API key client-side):
- `src/app/api/consent/route.ts` — accepts `{ device_id, consent_type, granted }`, forwards to MMS with server-side API key

### 3.2 Conditional Tracker Loading

**File:** `src/app/layout.tsx`

Replace unconditional tracker script (line 232-239) with:

```tsx
<CookieConsentBanner />
<ConditionalTracker />
```

**New file:** `src/components/ConditionalTracker.tsx` (Client Component)

- Reads `localStorage.cookie_consent`
- If `'granted'` → renders `<Script src="https://analytics.masem.at/tracker.js" ... />`
- If anything else → renders nothing
- Listens for storage event to react to banner interaction

---

## Phase 4: Privacy Policy Update (P0-B)

**File:** `src/app/privacy/page.tsx`

### 4.1 Section 1 — add "Free Governance Score Card" subsection

After "Free Governance Health Checks":

- Email address (for Score Card PDF delivery)
- DAO identifier and governance scores (for Score Card content)
- Optional marketing consent (for newsletter — opt-in only)
- IP address (rate limiting, consent audit trail)
- Data stored in masemIT Management System (api.masem.at, Austria, EU)

### 4.2 Section 2 — add row to legal basis table

| Score Card delivery | Performance of contract (Art. 6(1)(b)) |
| Score Card marketing consent | Consent (Art. 6(1)(a)) |
| Cookie analytics (masemIT tracker) | Consent (Art. 6(1)(a)) |

### 4.3 Section 3 — add MMS as data processor

| masemIT (MMS) | Lead & consent management | Austria (EU) | GDPR |

### 4.4 Section 8 — rewrite Analytics section

Current text claims only "Vercel Analytics (no cookies)" — this is **inaccurate**.

New text must cover:
- **Vercel Analytics** — privacy-first, no cookies, no consent required
- **masemIT Analytics** (analytics.masem.at) — self-hosted tracker, uses cookies, requires consent via cookie banner
- Consent banner reference: "You can manage your preferences via the cookie banner shown on first visit."

### 4.5 New Section: Cookies

Insert between current Section 7 (Security) and Section 8 (Analytics):

| Cookie | Purpose | Duration | Legal Basis |
|--------|---------|----------|-------------|
| `cs_session` | Authentication (Magic Link login) | 30 days | Art. 6(1)(b) — necessary for service |
| masemIT tracker | Analytics (page views, engagement) | Session | Art. 6(1)(a) — consent via banner |

### 4.6 Update "Last updated" to February 2026

---

## Phase 5: TellingCube Integration Guide (P2)

**New file:** `docs/integrations/mms-party-consent-integration-guide.md`

**Authors:** Mary (business requirements mapping) + Paige (technical documentation)

**Purpose:** Reusable integration guide for the MMS Party & Consent APIs, based on the ChainSights implementation. Meant for the TellingCube team so they don't start from zero.

**Contents:**
1. Overview: What MMS Parties & Consents provide
2. Architecture: API endpoints used, authentication, data flow
3. Implementation patterns (with code examples from ChainSights):
   - `createParty` with marketing consent
   - Party status lifecycle (lead → customer)
   - `unsubscribe_token` flow
   - Cookie consent via `device_id` + Consents API
   - Conditional tracker loading pattern
4. Privacy Policy template sections (copy-paste ready)
5. Checklist: What every MMS-integrated project needs
6. Lessons learned from ChainSights (edge cases, gotchas)

---

## Files Touched

| File | Change |
|------|--------|
| `src/lib/mms/analytics.ts` | Extend `createParty`, add `updatePartyStatus` |
| `src/app/api/score-card/route.ts` | Pass consent fields, use `unsubscribeToken` |
| `src/lib/email/index.ts` | Add unsubscribe link to Score Card email |
| `src/app/unsubscribe/page.tsx` | **NEW** — unsubscribe page |
| `src/app/api/unsubscribe/route.ts` | **NEW** — proxy to MMS unsubscribe endpoint |
| `src/app/api/webhooks/stripe/route.ts` | Add `createParty` + status upgrade |
| `src/components/CookieConsentBanner.tsx` | **NEW** — consent banner |
| `src/components/ConditionalTracker.tsx` | **NEW** — conditional tracker loader |
| `src/app/api/consent/route.ts` | **NEW** — proxy to MMS Consents API |
| `src/app/layout.tsx` | Replace unconditional tracker with conditional components |
| `src/app/privacy/page.tsx` | Add Score Card, MMS, cookies, masemIT tracker sections |
| `docs/integrations/mms-party-consent-integration-guide.md` | **NEW** — TellingCube guide |

---

## Edge Cases Discussed

| Scenario | Expected Behavior |
|----------|-------------------|
| User gets Score Card (lead) then buys Deep Dive | MMS find-or-create returns existing party, webhook PATCHes status to `customer`. Marketing consent preserved. |
| User buys Deep Dive first, then gets Score Card with marketing opt-in | `createParty` finds existing customer, adds marketing consent + generates `unsubscribe_token`. |
| User declines cookies then accepts later | localStorage update triggers tracker load. New consent record created in MMS (append-only). |
| User unsubscribes then re-subscribes via new Score Card | MMS `POST /v1/parties` with `marketing_consent: true` creates new consent record. Old revocation preserved in audit trail. |
| Stripe webhook timeout | `createParty` is fire-and-forget. Webhook returns 200 regardless. MMS failure is non-critical. |
