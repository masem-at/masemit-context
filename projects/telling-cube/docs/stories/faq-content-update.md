# Story: FAQ Content Update

**Status:** Ready for Development
**Priority:** Low (Content Polish)
**Estimated Effort:** 15 minutes

---

## Overview

Update the FAQ section on the landing page to reflect current product features. The existing FAQ references outdated scenarios (Bakery, Hotel, Tech Startup) and misses new features (CSV export, API access).

---

## File to Update

`components/landing/FAQSection.tsx`

---

## Updated FAQ Content

Replace the `faqs` array with the following:

```typescript
const faqs = [
  {
    question: "What is tellingCube?",
    answer: "tellingCube is an AI-powered tool that generates realistic, multi-departmental business data in minutes. It's designed for business trainers, professors, and consultants who need demo datasets for workshops and presentations. Unlike traditional data generation tools, tellingCube ensures that all data views (Sales, Finance) reconcile perfectly through event-sourced architecture."
  },
  {
    question: "How does multi-view consistency work?",
    answer: "tellingCube uses an event-sourced architecture that generates business events (like sales transactions, employee hires, inventory purchases) and then derives all departmental views from these events. This ensures that when Sales shows a certain revenue, Finance shows the same amount—all numbers reconcile perfectly, using visualizations inspired by IBCS© for AC (Actual), PY (Previous Year), PL (Plan), and FC (Forecast) comparisons."
  },
  {
    question: "Do I need an account?",
    answer: "No account is needed for the PoC version. Just click one of the company tier buttons to generate your data instantly."
  },
  {
    question: "What company types are available?",
    answer: "tellingCube offers three enterprise-relevant company tiers: Finance / Large Cap (major financial services companies), Industrials / Mid Cap (manufacturing and industrial firms), and Consumer Staples / Startup (consumer goods startups). Each generates realistic business data appropriate for that company size and sector."
  },
  {
    question: "Can I export the data?",
    answer: "Yes! Each view (Sales, Finance) has a CSV download button for easy export to Excel, Power BI, or other tools. We also provide a REST API for programmatic access—check the API Docs link on the results page."
  },
  {
    question: "Is my payment secure?",
    answer: "Yes, absolutely. All payments are processed securely through Stripe, a leading payment processor trusted by millions of businesses worldwide. We never store your payment information on our servers."
  },
  {
    question: "Can I get a refund?",
    answer: "Yes, we offer a 30-day money-back guarantee for founding members. If tellingCube doesn't meet your expectations, simply contact us at support@tellingcube.com within 30 days of purchase for a full refund—no questions asked."
  }
];
```

---

## Changes Summary

| Question | Change |
|----------|--------|
| "What is tellingCube?" | Removed HR/Controlling mentions (not yet implemented), simplified |
| "How does multi-view consistency work?" | Added "inspired by IBCS©" mention (AC, PY, PL, FC) |
| "Do I need an account?" | Simplified to "PoC version", removed founding member mention |
| "What scenarios are available?" | **Renamed** to "What company types are available?" - Updated to new tier system (Finance/Large Cap, Industrials/Mid Cap, Consumer Staples/Startup) |
| **NEW** "Can I export the data?" | Added question about CSV export and API access |
| Payment & Refund questions | Kept as-is |

---

## Acceptance Criteria

- [ ] FAQ content updated in `FAQSection.tsx`
- [ ] No TypeScript errors
- [ ] Landing page displays correctly
- [ ] All accordion items expand/collapse properly

---

## Notes

- Simple text replacement task
- No structural changes to the component
- Content aligned with current PoC features as of December 2024
