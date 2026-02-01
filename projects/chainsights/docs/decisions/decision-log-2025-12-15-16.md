# ChainSights Decision Log

## Session: December 15-16, 2025
**Participants:** Mario (Founder), BMAD Agents (Pixel Designer, Daniel Legal, Mary Analyst, Anna Billing, Dev team via Party Mode)

---

## Strategic Decisions

### D1: Two-Tier Pricing Model
**Decision:** Launch with two pricing tiers instead of single price
- **Deep Dive:** €49 (last 10 proposals, 24h delivery)
- **Governance Audit:** €149 (full history + trends, 48h delivery)

**Rationale:**
- Allows customers to start small and upgrade
- Deep Dive captures more value from serious governance leads
- "Not sure which? Start with Standard" reduces decision friction

**Implementation:** Checkout API, Stripe metadata, orders table schema updated

---

### D2: Logo Concept - "The Cluster"
**Decision:** Selected cluster/convergence concept over alternatives (lens, fingerprint, bridge)

**Rationale:**
- Visually represents wallet identity resolution (many wallets → one entity)
- Works at all sizes (icon to full logo)
- Aligns with tagline "Wallets lie. We don't."

**Assets Created:**
- `public/images/logo.svg` (horizontal)
- `public/images/logo-full.svg` (full)
- `public/images/icon.svg` (favicon)
- `docs/design/chainsights-logo-guidelines.md`

---

### D3: Landing Page Copy v1.1
**Decision:** Complete rewrite focusing on problem/solution narrative

**Key Changes:**
- Hero: Added ENS stat ("302 voters out of 142,908 followers")
- Problem section: "The uncomfortable truth" bullet list
- Solution section: Whale vs Delegate concentration distinction (key differentiator)
- FAQ section: 6 questions addressing common objections
- Sample insight: ENS quote showing nuanced analysis

**Source:** `docs/_masemIT/ChainSights_Landing_Page_v1.1.md`

---

### D4: GDPR Compliance Approach
**Decision:** Implement practical GDPR compliance for MVP launch

**Key Elements:**
1. Privacy policy with full disclosures (data recipients, retention, rights)
2. Admin-only GDPR endpoints (export/delete) for handling requests
3. Orders anonymized but retained 7 years (Austrian tax law)
4. Reports and share rewards fully deletable
5. DPAs to be signed with all processors (manual task)

**Documents Created:**
- `docs/legal/gdpr-gap-analysis.md`
- `docs/legal/gdpr-remediation-plan.md`
- `docs/legal/ropa-template.md`

**Endpoints Created:**
- `POST /api/admin/gdpr/export`
- `POST /api/admin/gdpr/delete`

---

### D5: Security - Password Hashing
**Decision:** Implement bcrypt password hashing for admin authentication

**Approach:**
- Support both `ADMIN_PASSWORD_HASH` (preferred) and `ADMIN_PASSWORD` (backward compat)
- 10 rounds bcrypt
- Async validation

**Migration Path:** Generate hash, add to Vercel env, remove plain password

---

## Product Decisions (from Party Mode Feedback)

### D6: Report Structure - Executive Summary First
**Decision:** Move executive summary to page 1 of PDF report

**Rationale:** Governance leads are busy; key findings should be immediately visible

---

### D7: Report Footer - QR Code
**Decision:** Add QR code linking to chainsights.one in report footer

**Rationale:** Makes sharing and follow-up easy when report is printed/shared

---

### D8: Scope Disclaimer
**Decision:** Add clear scope statement to reports

**Text:** "Analysis based on Snapshot governance data. On-chain token transfers and multisig activity not included in this version."

**Rationale:** Sets expectations, avoids overpromising, opens door for future upsells

---

### D9: Share & Save Marketing
**Decision:** €20 cashback for honest Twitter share (mention @ChainSights_one)

**Rationale:**
- Reduces effective price to €79
- Generates social proof
- "Praise, critique, whatever — we honor it" builds trust

---

## Technical Decisions

### D10: Database Schema - Tier Column
**Decision:** Add `tier` enum column to orders table

**Values:** `standard`, `deep_dive`
**Default:** `standard`

---

### D11: Admin UX - Home Link
**Decision:** Add home link button to admin dashboard header

**Rationale:** Easy navigation between admin and public site

---

## Deferred Decisions

### Invoicing Integration
**Status:** Pending consultation with Steuerberater
**Questions:**
- Registrierkasse requirements for digital services?
- Invoice numbering format?
- OSS registration timing?

**Owner:** Mario + Anna (Billing Agent)

---

### Rate Limiting
**Status:** Deferred to post-launch
**Priority:** Low (single admin, low traffic expected initially)

---

## Reference Commits

| Commit | Description |
|--------|-------------|
| `5bdb183` | Initial commit |
| `df96eda` | Fix admin auth cookie path and logout |
| `bd47717` | Add header to legal pages, Twitter to footer |
| `f76bde3` | Landing page v1.1, privacy policy GDPR |
| `59538a2` | Two-tier pricing checkout API |
| `26f090c` | Tier in orders, bcrypt password |
| `0dfe639` | GDPR endpoints, admin home link |

---

## Next Steps (as of Dec 16)

1. [ ] Sign DPAs with processors (Stripe, Vercel, Neon, Resend, Anthropic, Upstash)
2. [ ] Set `ADMIN_PASSWORD_HASH` in Vercel, remove plain password
3. [ ] Resolve invoicing requirements with Steuerberater
4. [ ] First customer outreach (Elena at ENS?)

---

*Document created by Mary (Business Analyst) - December 16, 2025*
