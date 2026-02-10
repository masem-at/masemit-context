# DEV TICKET: Pricing Page (/pricing)

> **⚠️ PRICING OUTDATED** — DAO Matrix is FREE since 2026-02-05. €19/mo, €99/yr, and €199/yr are all deprecated. Current: Deep Dive €49, Governance Audit €149. See `docs/project_context.md`.

**Priority:** Medium
**Effort:** Small
**Dependencies:** None (fully independent, no auth needed)

---

## Context

ChainSights hat aktuell keine dedizierte Pricing-Seite. Preise sind nur sichtbar wenn ein CTA ausgeführt wird (z.B. beim Report-Bestellen oder Matrix-Abo). Das erschwert Transparenz, Kommunikation und SEO.

Gründe für eine `/pricing` Route:
- Transparenz schafft Vertrauen (im Web3-Space besonders wichtig)
- SEO-Vorteil ("chainsights pricing" als Suchbegriff)
- Referenz-Link für Gespräche, DMs, Discord, Forum-Posts
- Klarer Vergleich aller Optionen auf einen Blick

---

## Current Pricing Structure

### One-Time Reports

| Tier | Price | Target Audience |
|------|-------|-----------------|
| **Quick Check** | Free (email required) | Curious visitors, lead gen |
| **Deep Dive** | €49 | DAO contributors, small teams |
| **Governance Audit** | €149 | DAO core teams, serious governance leads |

### Subscriptions

| Product | Price | Target Audience |
|---------|-------|-----------------|
| **DAO Matrix Access** | €19/month or €99/year | Researchers, VCs, analysts |

### Future (show as "Coming Soon" or omit)

| Product | Price | Target Audience |
|---------|-------|-----------------|
| **API Access** | €49/month | Tooling platforms, aggregators |
| **Verified Badge** | €199/year | DAOs claiming their listing |

---

## Page Structure

### Route: `/pricing`

### Layout

```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│  Headline: "Transparent Pricing. No Hidden Fees."           │
│  Subline: "From free governance checks to full audits."     │
│                                                             │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  SECTION 1: One-Time Reports                                │
│                                                             │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │  Quick Check  │  │  Deep Dive   │  │  Gov Audit   │      │
│  │    FREE       │  │    €49       │  │    €149      │      │
│  │              │  │              │  │  ★ Popular   │      │
│  │  • GVS Score  │  │  Everything  │  │  Everything  │      │
│  │  • Components │  │  in QC plus: │  │  in DD plus: │      │
│  │  • Category   │  │  • AI Anal.  │  │  • Hist.     │      │
│  │    comparison │  │  • Recommend │  │  • Peers     │      │
│  │              │  │  • Benchmark │  │  • DGI Bench  │      │
│  │              │  │  • PDF       │  │  • Full PDF  │      │
│  │              │  │              │  │              │      │
│  │  [Check DAO] │  │ [Get Report] │  │ [Get Audit]  │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
│                                                             │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  SECTION 2: DAO Matrix (Subscription)                       │
│                                                             │
│  ┌──────────────────────┐  ┌──────────────────────┐        │
│  │  Monthly              │  │  Annual (save 57%)    │        │
│  │  €19/month            │  │  €99/year             │        │
│  │                      │  │  = €8.25/month        │        │
│  │  • 44 DAOs tracked   │  │  • Everything monthly │        │
│  │  • Daily updates     │  │  • 2 months free      │        │
│  │  • All metrics       │  │                      │        │
│  │  • Sort & filter     │  │                      │        │
│  │  • Detail pages      │  │                      │        │
│  │                      │  │                      │        │
│  │  [Subscribe Monthly] │  │  [Subscribe Annual]   │        │
│  └──────────────────────┘  └──────────────────────┘        │
│                                                             │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  SECTION 3: FAQ (expandable)                                │
│                                                             │
│  ▸ What's included in a Deep Dive vs Governance Audit?      │
│  ▸ Can I upgrade from Deep Dive to Governance Audit?        │
│  ▸ How do I cancel my Matrix subscription?                  │
│  ▸ Do you offer refunds?                                    │
│  ▸ What DAOs can I analyze?                                 │
│  ▸ Is there a free trial for the Matrix?                    │
│                                                             │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  FOOTER CTA: "Not sure? Try a free Quick Check first."     │
│  [Check Your DAO — Free]                                    │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## CTA Destinations

| Button | Destination |
|--------|-------------|
| Check DAO (Quick Check) | `/` or Quick Check modal (existing flow) |
| Get Report (Deep Dive) | Existing report order flow with `tier=deep_dive` |
| Get Audit (Governance Audit) | Existing report order flow with `tier=governance_audit` |
| Subscribe Monthly | Stripe Checkout (existing Matrix monthly product) |
| Subscribe Annual | Stripe Checkout (existing Matrix annual product) |

---

## Navigation Integration

Add "Pricing" to the main navigation bar:

```
Logo | [Rankings] [DGI] [Matrix] [Pricing] | [Login]
```

---

## SEO Requirements

- Page title: "Pricing — ChainSights | DAO Governance Analytics"
- Meta description: "Transparent pricing for DAO governance reports and analytics. Free quick checks, deep dive reports from €49, and full governance audits."
- Open Graph: Standard ChainSights branding
- Canonical URL: `https://chainsights.one/pricing`

---

## Design Notes

- Follow existing ChainSights design system (dark teal #093236, accent cyan #009BB1)
- Report tier cards: 3 columns on desktop, stacked on mobile
- Subscription cards: 2 columns on desktop, stacked on mobile
- "Popular" badge on Governance Audit (highest margin, most value)
- Annual plan should visually emphasize savings ("Save 57%", "2 months free")
- No need for toggle between monthly/annual — show both side by side

---

## Acceptance Criteria

- [ ] `/pricing` route exists and is accessible
- [ ] All current products displayed with correct prices
- [ ] CTAs link to correct checkout/order flows
- [ ] Navigation includes "Pricing" link
- [ ] Responsive: works on desktop, tablet, mobile
- [ ] FAQ section with expandable items
- [ ] SEO meta tags set correctly
- [ ] Page tracked with analytics (pricing_view event)

---

## Files to Create/Touch

1. `src/app/pricing/page.tsx` — New page component
2. Navigation component — Add "Pricing" link
3. Optional: Shared pricing data config (to keep prices in sync across pages)

---

## Out of Scope

- Starter-Tier (geparkt, wird nach Validierung entschieden)
- API Access / Verified Badge pricing (Future, nicht anzeigen oder nur "Coming Soon")
- Coupon/discount system
- Account/login functionality (separate ticket)
