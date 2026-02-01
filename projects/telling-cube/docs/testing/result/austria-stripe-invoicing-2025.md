# Austria (EPU) + Stripe SaaS/API Plans: Invoicing & VAT Checklist (as of 14 Dec 2025)

This is a practical, **non-legal, non-tax-advice** checklist for an Austrian sole proprietor (EPU) selling SaaS/API plans (e.g., **Unlimited** subscription and **Lifetime** access) with **Stripe** and sending emails via **Resend**.

---

## 1) Stripe vs. you: who is responsible for invoices?

**Stripe processes payments**. In most setups, **you** remain responsible for:
- deciding whether VAT is charged,
- issuing compliant invoices (especially for B2B),
- storing invoices/records for the required retention period.

---

## 2) First decision: are you an Austrian “small business” (Kleinunternehmer) in 2025?

Austria’s small-business VAT regime changed from **1 Jan 2025**:
- The annual turnover limit is **€55,000 (gross)**.  
- There is a **10% tolerance**:  
  - if you exceed the €55,000 limit by **no more than 10%**, you may continue issuing invoices **without VAT until year-end**;  
  - if you exceed by **more than 10%**, **further invoices from that point** become VAT-taxable.  

Sources: WKO small business VAT page; USP news item.  
- https://www.wko.at/steuern/kleinunternehmerregelung-umsatzsteuer  
- https://www.usp.gv.at/aktuelles/newsliste/kleinunternehmerregelung-ab-2025.html

**Practical implication**
- If you are a Kleinunternehmer: **do not show Austrian VAT** on invoices; add a note that the supply is VAT-exempt under the small-business regime.
- If you are VAT-registered (or opt-in): invoice with VAT where applicable and follow the normal VAT rules.

---

## 3) When must you issue invoices?

### B2B (and legal entities)
You must issue an invoice if you supply:
- another **entrepreneur for their business**, or
- a **legal entity** (even if it is not an entrepreneur).

Deadline: you must comply **within 6 months** after the supply is performed.

Source (Austrian law § 11 UStG):  
- https://www.jusline.at/gesetz/ustg/paragraf/11

### B2C (private individuals)
Full formal invoice features are **generally not required** for invoices to private individuals, but issuing automated receipts/invoices is still common and helpful.

Source: WKO “Erfordernisse einer Rechnung”.  
- https://www.wko.at/steuern/rechnung-richtig-ausstellen

---

## 4) What must be on an Austrian invoice (core items)

For “full” invoices (especially relevant for B2B / input VAT deduction), include at least:
- your business name and address,
- customer name and address,
- invoice date,
- **sequential invoice number**,
- description of the service (and quantity/extent),
- **date or period of supply** (e.g., month/year for subscriptions),
- net amount (and discounts if any),
- VAT rate and VAT amount (if VAT applies), or a VAT-exemption note (if not),
- your VAT ID (UID), if applicable.

Source: WKO “Erfordernisse einer Rechnung” (and Austrian § 11 UStG overview via USP).  
- https://www.wko.at/steuern/rechnung-richtig-ausstellen  
- https://www.usp.gv.at/themen/steuern-finanzen/umsatzsteuer-ueberblick/weitere-informationen-zur-umsatzsteuer/vorsteuerabzug-und-rechnung/formerfordernisse.html

**Tip for your plans**
- Subscription: show a **billing / service period** (e.g., “Unlimited Plan – Dec 2025”).
- Lifetime: treat as a one-time supply (“Lifetime access granted on YYYY-MM-DD”) and show the supply date.

---

## 5) EU customers: OSS + the €10,000 threshold (B2C digital services)

If you sell to **EU private consumers** outside Austria, VAT can become due in the customer’s EU Member State.

The European Commission explains an EU-wide **€10,000 threshold**:
- **below** €10,000 (cross-border B2C distance sales and certain digital services / TBE), VAT **may remain** in the Member State where you are established;
- **above** €10,000, VAT is generally due in the customer’s Member State.

Source (EU Commission OSS portal):  
- https://vat-one-stop-shop.ec.europa.eu/index_en

### OSS (One-Stop Shop)
OSS is the EU portal that allows you to declare and pay VAT for eligible cross-border B2C supplies without registering in every Member State.

Source (USP, EN):  
- https://www.usp.gv.at/en/themen/steuern-finanzen/umsatzsteuer-ueberblick/weitere-informationen-zur-umsatzsteuer/umsaetze-mit-auslandsbezug/Umsatzsteuer-One-Stop-Shop.html

### OSS record keeping (10 years)
If you use OSS, the EU Commission states that records must be kept for **10 years from the end of the year** in which the transaction occurred.

Source (EU Commission OSS – record keeping):  
- https://vat-one-stop-shop.ec.europa.eu/one-stop-shop/record-keeping-and-audits-oss_en

---

## 6) Customers outside the EU (third countries): what changes?

When you sell to customers in **non-EU countries** (e.g., USA, UK, Switzerland, Norway, Australia), there are two separate questions:

1. **Is Austrian/EU VAT due?** (often: no, because the place of supply is outside Austria/EU)  
2. **Could you become tax-registered in the customer’s country?** (often: yes, depending on that country’s rules and thresholds)

### 6.1 Austrian/EU VAT perspective (place of supply)

For cross-border **services**, Austria follows the general “place of supply” approach:
- **B2B:** place of supply is generally where the **business customer is established** (recipient principle).  
- **B2C:** special rules apply for telecom/broadcasting/electronically supplied services; for customers outside the EU, the place of taxation can be outside the EU depending on the rule in the VAT Directive.

Sources:
- USP “Grenzüberschreitende Dienstleistungen”: https://www.usp.gv.at/themen/steuern-finanzen/umsatzsteuer-ueberblick/weitere-informationen-zur-umsatzsteuer/umsaetze-mit-auslandsbezug/grenzueberschreitende-dienstleistungen.html  
- WKO “Dienstleistungen an ausländische Unternehmer (B2B)”: https://www.wko.at/steuern/b2b-dienstleistungen-auslaendische-unternehmer  
- European Commission (VAT Directive – place of taxation): https://taxation-customs.ec.europa.eu/taxation/vat/vat-directive/place-taxation_en

**Practical invoicing outcome (typical):**
- Many non-EU sales end up **not taxable in Austria** → you invoice **without Austrian VAT** and include a short note such as “place of supply outside Austria / not taxable in Austria”.
- If you become locally registered in a third country, you may need to show **that country’s VAT/GST/Sales Tax** and comply with local invoice rules.

### 6.2 Third-country local taxes (examples you should watch)

These are examples of common “gotchas” for SaaS/digital services. Exact rules differ by country and may change.

#### United Kingdom (UK)
The UK has specific rules for **digital services supplied to private consumers** (B2C).  
Source (GOV.UK guidance): https://www.gov.uk/guidance/the-vat-rules-if-you-supply-digital-services-to-private-consumers

#### Norway (VOEC)
Norway’s tax authority states that **foreign providers of remotely deliverable services** (and low value goods) may be **obliged to collect and pay VAT** under the VOEC scheme.  
Source (Norwegian Tax Administration): https://www.skatteetaten.no/en/business-and-organisation/vat-and-duties/vat/foreign/e-commerce-voec/

#### Australia (GST on imported services & digital products)
Australia applies GST to sales by **non-resident businesses** to Australian consumers of certain **imported services and digital products**.  
Source (ATO): https://www.ato.gov.au/businesses-and-organisations/international-tax-for-business/gst-for-non-resident-businesses/gst-on-imported-services-and-digital-products  
General GST registration threshold for overseas businesses is commonly stated as **A$75,000** (check current rules and whether platforms/EDPs apply).  
Source (business.gov.au overview): https://business.gov.au/finance/tax/international-tax

#### Switzerland (VAT / MWST for foreign companies)
Switzerland’s Federal Tax Administration explains that foreign companies may be VAT-liable; businesses with annual turnover **below CHF 100,000** from taxable/zero-rated supplies are generally exempt (unless they opt in).  
Source (Swiss FTA): https://www.estv.admin.ch/estv/en/home/value-added-tax/vat-tax-liability/foreign-companies.html

#### United States (Sales tax economic nexus, state-by-state)
In the US, sales tax is **state-level**; many states have “economic nexus” thresholds for remote sellers (influenced by the Wayfair decision). Thresholds vary by state and can be based on revenue and/or transactions.  
Sources:
- Streamlined Sales Tax (remote seller guidance): https://www.streamlinedsalestax.org/for-businesses/remote-seller-faqs/remote-seller-state-guidance  
- Sales Tax Institute (state-by-state nexus chart): https://www.salestaxinstitute.com/resources/economic-nexus-state-guide

### 6.3 Implementation tips for Stripe checkout
To handle third-country scenarios cleanly, collect enough data to classify the transaction:
- **Country + billing address**
- **B2B vs B2C indicator** (company name; in the EU: VAT ID; outside EU: other business evidence)
- Store evidence used to determine location and customer status (useful for audits and disputes)

If you start getting meaningful revenue in specific third countries, consider enabling a tax engine (e.g., Stripe Tax) and/or using a compliance partner for registrations and filings.

---

## 7) EU B2B sales: Reverse Charge (common pattern)

For many cross-border B2B services in the EU, **Reverse Charge** applies:
- you invoice **net** (no Austrian VAT),
- include the customer’s VAT ID,
- include a statement indicating the VAT liability shifts to the recipient (e.g., “Reverse charge” / “VAT liability of the recipient”).

Sources:  
- WKO Reverse Charge FAQ (invoice specifics): https://www.wko.at/steuern/umsatzsteuer-rechnung-faq  
- USP Reverse Charge overview: https://www.usp.gv.at/themen/steuern-finanzen/umsatzsteuer-ueberblick/weitere-informationen-zur-umsatzsteuer/umsaetze-mit-auslandsbezug/reverse-charge.html

---

## 8) How long do you need to store invoices and records?

### Austria (general retention)
Austria generally requires retaining accounting records and relevant business documents for **7 years**, with the period running from the end of the relevant calendar year.

Sources:
- WKO retention overview: https://www.wko.at/steuern/aufbewahrungspflichten  
- USP retention page: https://www.usp.gv.at/themen/steuern-finanzen/steuerliche-gewinnermittlung/weitere-informationen-zur-steuerlichen-gewinnermittlung/betriebliches-rechnungswesen/aufbewahrungspflicht.html  
- Law reference (BAO § 132): https://www.jusline.at/gesetz/bao/paragraf/132

### OSS (special retention)
If you use OSS, keep records for **10 years** (EU rule) — see Section 5.

---

## 9) Practical implementation checklist (Stripe + Resend)

**Checkout data you should collect**
- customer name + billing address,
- country (for VAT place-of-supply logic),
- for B2B: company name + **VAT ID (UID)**.

**Stripe configuration**
- enable invoices / receipts (or your own invoice generator),
- configure a sequential invoice numbering scheme,
- store invoice PDFs and underlying transaction data.

**Automation**
- after successful payment, send a welcome email via Resend that includes:
  - the invoice PDF (or secure link),
  - VAT status summary (e.g., “Reverse charge”, “VAT-exempt small business”, etc.).

**Compliance hygiene**
- keep a simple audit trail: plan purchased, price, timestamps, customer country evidence, VAT ID validation status, and the exact invoice issued.

---

## References (primary)
- WKO – Invoice requirements: https://www.wko.at/steuern/rechnung-richtig-ausstellen  
- Austria – Small business VAT regime (from 1 Jan 2025): https://www.wko.at/steuern/kleinunternehmerregelung-umsatzsteuer  
- Austrian UStG § 11 (invoicing duty + 6-month deadline): https://www.jusline.at/gesetz/ustg/paragraf/11  
- EU Commission – OSS portal + €10,000 threshold: https://vat-one-stop-shop.ec.europa.eu/index_en  
- EU Commission – OSS record keeping (10 years): https://vat-one-stop-shop.ec.europa.eu/one-stop-shop/record-keeping-and-audits-oss_en  
- USP (EN) – OSS overview: https://www.usp.gv.at/en/themen/steuern-finanzen/umsatzsteuer-ueberblick/weitere-informationen-zur-umsatzsteuer/umsaetze-mit-auslandsbezug/Umsatzsteuer-One-Stop-Shop.html  
- Austria retention (7 years): WKO https://www.wko.at/steuern/aufbewahrungspflichten ; USP https://www.usp.gv.at/themen/steuern-finanzen/steuerliche-gewinnermittlung/weitere-informationen-zur-steuerlichen-gewinnermittlung/betriebliches-rechnungswesen/aufbewahrungspflicht.html ; BAO § 132 https://www.jusline.at/gesetz/bao/paragraf/132
- Place of supply (services) / cross-border services: USP https://www.usp.gv.at/themen/steuern-finanzen/umsatzsteuer-ueberblick/weitere-informationen-zur-umsatzsteuer/umsaetze-mit-auslandsbezug/grenzueberschreitende-dienstleistungen.html ; WKO https://www.wko.at/steuern/b2b-dienstleistungen-auslaendische-unternehmer ; EU Commission https://taxation-customs.ec.europa.eu/taxation/vat/vat-directive/place-taxation_en
- UK digital services VAT (B2C): GOV.UK https://www.gov.uk/guidance/the-vat-rules-if-you-supply-digital-services-to-private-consumers
- Norway VOEC (remote services): Norwegian Tax Administration https://www.skatteetaten.no/en/business-and-organisation/vat-and-duties/vat/foreign/e-commerce-voec/
- Australia GST on imported digital services/products: ATO https://www.ato.gov.au/businesses-and-organisations/international-tax-for-business/gst-for-non-resident-businesses/gst-on-imported-services-and-digital-products ; business.gov.au https://business.gov.au/finance/tax/international-tax
- Switzerland VAT for foreign companies: Swiss FTA https://www.estv.admin.ch/estv/en/home/value-added-tax/vat-tax-liability/foreign-companies.html
- USA sales tax economic nexus (overview): Streamlined Sales Tax https://www.streamlinedsalestax.org/for-businesses/remote-seller-faqs/remote-seller-state-guidance ; Sales Tax Institute https://www.salestaxinstitute.com/resources/economic-nexus-state-guide

---

**Reminder:** VAT/OSS details can depend on your exact setup (customer type, place-of-supply evidence, VAT-ID validation, product characterization). If you’re close to thresholds or already selling internationally, a short Austrian tax advisor check is usually worth it.
