# masemIT e.U. — VAT & Invoicing Requirements

**Entity:** masemIT e.U.
**Location:** Vienna, Austria
**VAT Status:** Regelbesteuert (voluntarily VAT-registered)
**Applies To:** All products (ChainSights, TellingCube, future projects)

**Last Updated:** 2025-12-16
**Owner:** Mario + Anna (Billing Agent)

---

## Key Decision: Gross Pricing

**All prices are VAT-inclusive (gross).**

| Product | Advertised Price | Netto | 20% USt | Customer Pays |
|---------|------------------|-------|---------|---------------|
| ChainSights Deep Dive | €49 | €40.83 | €8.17 | €49 |
| ChainSights Governance Audit | €149 | €124.17 | €24.83 | €149 |
| TellingCube [plans] | [TBD] | [TBD] | [TBD] | [TBD] |

**Rationale:** Cleaner UX for B2C customers, price = what they pay.

---

## VAT Treatment by Customer Type

### 1. Austrian Customers (B2C or B2B)

**→ 20% Austrian VAT applies**

| Field | Value |
|-------|-------|
| VAT Rate | 20% |
| Invoice shows | Netto + USt breakdown |
| You remit | USt to Finanzamt via UVA |

**Example Invoice Line:**
```
ChainSights Deep Dive Report        €40.83
USt 20%                             €8.17
────────────────────────────────────────────
Gesamt                             €49.00
```

---

### 2. EU B2B Customer (valid VAT ID)

**→ Reverse Charge, 0% VAT**

| Field | Value |
|-------|-------|
| VAT Rate | 0% |
| Invoice shows | Netto only (€49 or €149) |
| Invoice note | Reverse Charge statement |
| Requirement | Validate VAT ID via VIES |

**Invoice Note (required):**
> *"Reverse Charge – Steuerschuldnerschaft des Leistungsempfängers gem. Art. 196 MwStSystRL"*

**VIES Validation:** https://ec.europa.eu/taxation_customs/vies/

**Important:** Customer pays the gross price equivalent (€49/€149) but receives a netto invoice. The "VAT portion" becomes additional revenue for you in this scenario.

---

### 3. EU B2C Customer (no VAT ID)

**→ Charge VAT (Austrian rate until OSS threshold)**

| Field | Value |
|-------|-------|
| VAT Rate | 20% Austrian (below €10k threshold) |
| Invoice shows | Netto + USt breakdown |
| Threshold | €10,000/year EU-wide B2C cross-border |

**Below €10,000 threshold:**
- Charge Austrian 20% VAT
- Treat same as Austrian customer

**Above €10,000 threshold:**
- Must register for OSS
- Charge destination country VAT rate
- Or use Stripe Tax to automate

**Action:** Track EU B2C sales by country. Register for OSS when approaching €8,000.

---

### 4. Non-EU Customer (USA, UK, CH, etc.)

**→ No Austrian VAT (not taxable in Austria)**

| Field | Value |
|-------|-------|
| VAT Rate | 0% (not applicable) |
| Invoice shows | Gross amount as netto |
| Invoice note | Place of supply statement |

**Invoice Note:**
> *"Nicht steuerbar – Leistungsort außerhalb der EU / Not subject to Austrian VAT – place of supply outside EU"*

**Important:** Customer pays €49/€149. Since no VAT applies, this is 100% revenue (no VAT to remit).

**Watch for:** Local tax obligations in customer's country if you scale significantly (see third-country rules in `austria-stripe-invoicing-2025.md`).

---

## Invoice Requirements (§ 11 UStG)

Every invoice must include:

| # | Requirement | Example |
|---|-------------|---------|
| 1 | Your business name & address | masemIT e.U., [Address], Vienna, Austria |
| 2 | Customer name & address | [From checkout] |
| 3 | Your UID number | ATUxxxxxxxx |
| 4 | Customer UID (B2B >€10k) | [If provided] |
| 5 | Sequential invoice number | CS-2025-0001 |
| 6 | Invoice date | 2025-12-16 |
| 7 | Service date | 2025-12-16 |
| 8 | Service description | ChainSights Deep Dive Governance Report |
| 9 | Netto amount | €40.83 |
| 10 | VAT rate | 20% |
| 11 | VAT amount | €8.17 |
| 12 | Gross amount | €49.00 |

**For Reverse Charge:** Omit VAT rate/amount, add RC statement, include customer UID.

**For Non-EU:** Omit VAT rate/amount, add place-of-supply statement.

---

## Invoice Numbering

**Format:** `CS-YYYY-NNNN` (ChainSights) / `TC-YYYY-NNNN` (TellingCube)

| Rule | Description |
|------|-------------|
| Sequential | No gaps allowed |
| Unique | Never reuse a number |
| Per year | Reset to 0001 each January (optional) |
| Cancelled invoice | Issue Stornorechnung, don't skip number |

**Implementation:** Add `invoice_number` column to orders table, auto-increment per project prefix.

---

## Sample Invoices

### Austrian/EU B2C Customer
```
RECHNUNG / INVOICE
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

masemIT e.U.
[Your Address]
1XXX Vienna, Austria
UID-Nr: ATUxxxxxxxx

An / To:
Max Mustermann
[Address]
Austria

Rechnungsnummer: CS-2025-0001
Rechnungsdatum: 2025-12-16
Leistungsdatum: 2025-12-16

───────────────────────────────────────────────────────────
Pos  Beschreibung                              Netto (EUR)
───────────────────────────────────────────────────────────
1    ChainSights Standard Governance Report        82.50
     DAO: ens.eth
     Analyse: Letzte 10 Proposals
───────────────────────────────────────────────────────────
     Zwischensumme                                  82.50
     USt 20%                                        16.50
     ═════════════════════════════════════════════════════
     Gesamtbetrag                                   99.00

Zahlung erhalten via Stripe am 2025-12-16.
```

### EU B2B Customer (Reverse Charge)
```
RECHNUNG / INVOICE
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

masemIT e.U.
[Your Address]
1XXX Vienna, Austria
UID-Nr: ATUxxxxxxxx

An / To:
Example GmbH
[Address]
Germany
UID-Nr: DE123456789

Rechnungsnummer: CS-2025-0002
Rechnungsdatum: 2025-12-16
Leistungsdatum: 2025-12-16

───────────────────────────────────────────────────────────
Pos  Beschreibung                              Netto (EUR)
───────────────────────────────────────────────────────────
1    ChainSights Standard Governance Report        99.00
     DAO: ens.eth
     Analysis: Last 10 proposals
───────────────────────────────────────────────────────────
     Gesamtbetrag                                   99.00

Reverse Charge – Steuerschuldnerschaft des
Leistungsempfängers gem. Art. 196 MwStSystRL.

Payment received via Stripe on 2025-12-16.
```

### Non-EU Customer
```
INVOICE
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

masemIT e.U.
[Your Address]
1XXX Vienna, Austria

To:
John Smith
[Address]
United States

Invoice No: CS-2025-0003
Invoice Date: 2025-12-16
Service Date: 2025-12-16

───────────────────────────────────────────────────────────
Item                                          Amount (EUR)
───────────────────────────────────────────────────────────
ChainSights Standard Governance Report             99.00
DAO: ens.eth
Analysis: Last 10 proposals
───────────────────────────────────────────────────────────
Total                                              99.00

Not subject to Austrian VAT – place of supply outside EU.

Payment received via Stripe on 2025-12-16.
```

---

## Retention Requirements

| Record Type | Retention Period | Source |
|-------------|------------------|--------|
| Invoices (general) | 7 years | Austrian BAO § 132 |
| OSS records | 10 years | EU OSS rules |
| Stripe transaction data | 7 years minimum | Match invoice retention |

---

## Implementation Checklist

### Database Changes
- [ ] Add `invoice_number` (sequential) to orders table
- [ ] Add `customer_country` to orders table
- [ ] Add `customer_vat_id` to orders table (nullable)
- [ ] Add `vat_rate` to orders table
- [ ] Add `vat_amount_cents` to orders table

### Checkout Flow
- [ ] Collect billing country
- [ ] Optional: Collect VAT ID for B2B
- [ ] Validate VAT ID via VIES if provided
- [ ] Calculate correct VAT based on customer type

### Invoice Generation
- [ ] PDF template with all required fields
- [ ] Logic for different invoice types (AT, EU B2B, EU B2C, Non-EU)
- [ ] Attach to delivery email via Resend

### Stripe Configuration
- [x] Enable Stripe Tax (€0.50/transaction) ✓ Implemented 2025-12-16
- [ ] Store invoice_number in metadata
- [ ] Store VAT details in metadata

---

## Stripe Tax Setup

**Status:** ✓ Code implemented, Dashboard setup required

### Code Changes (Completed)
The checkout API (`src/app/api/checkout/route.ts`) now includes:
- `automatic_tax: { enabled: true }` — Enables Stripe Tax
- `billing_address_collection: 'required'` — Collects address for tax calculation
- `tax_id_collection: { enabled: true }` — Allows B2B customers to provide VAT ID
- `tax_behavior: 'inclusive'` — Prices include VAT (€49/€149 gross)
- `customer_creation: 'always'` — Creates customer record for invoicing

### Stripe Dashboard Setup (Required)
Complete these steps in Stripe Dashboard:

1. **Enable Stripe Tax**
   - Go to: [Stripe Dashboard > Settings > Tax](https://dashboard.stripe.com/settings/tax)
   - Click "Get started with Stripe Tax"

2. **Set Your Business Address**
   - Add masemIT e.U. address in Vienna, Austria
   - This determines your tax origin

3. **Add Your VAT Registration**
   - Add Austrian VAT ID: ATUxxxxxxxx
   - This enables reverse charge handling for EU B2B

4. **Configure Tax Behavior**
   - Default tax behavior: "Inclusive" (prices include tax)
   - This matches our gross pricing model

5. **Enable Automatic Tax Calculation**
   - Toggle on automatic tax for Checkout sessions

### What Stripe Tax Handles
- ✓ VAT ID validation via VIES
- ✓ Correct rate calculation by country
- ✓ Reverse charge for EU B2B
- ✓ Zero-rate for non-EU
- ✓ Tax reporting for UVA
- ✓ OSS threshold monitoring

**Cost:** €0.50 per transaction (~1% at €49-149 prices)

---

## References

- Austrian UStG § 11 (Invoice requirements): https://www.jusline.at/gesetz/ustg/paragraf/11
- WKO Invoice requirements: https://www.wko.at/steuern/rechnung-richtig-ausstellen
- EU VIES VAT validation: https://ec.europa.eu/taxation_customs/vies/
- EU OSS portal: https://vat-one-stop-shop.ec.europa.eu/index_en
- See also: `docs/_masemIT/austria-stripe-invoicing-2025.md`

---

## Disclaimer

This document is practical guidance based on Austrian tax law research. It does not constitute legal or tax advice. For complex scenarios or when approaching thresholds, consult your Steuerberater.

---

*Document created by Anna Hellinger (Billing Agent) — 2025-12-16*
