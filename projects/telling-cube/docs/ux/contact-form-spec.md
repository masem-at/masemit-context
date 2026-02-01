# Contact Form - UX Specification

**Author:** Sally (UX Lead)
**Date:** December 2024
**Status:** Ready for Review

---

## 1. Overview

Replace the current `mailto:` contact links with a proper contact form that:
- Provides a better user experience
- Captures valuable user insights
- Stores inquiries for tracking and response management

---

## 2. Page Location & Navigation

### New Route
- **URL:** `/contact`
- **Page Title:** "Contact Us | tellingCube"

### Navigation Updates
1. **Footer:** Change "Contact Support" mailto link → Link to `/contact`
2. **What's Next page:** Change "Email Us Your Ideas" mailto → Link to `/contact`
3. **Header:** No change (keep navigation minimal)

---

## 3. Form Layout

### Page Structure
```
┌─────────────────────────────────────────────────────────┐
│  Header (existing)                                      │
├─────────────────────────────────────────────────────────┤
│                                                         │
│     ┌─────────────────────────────────────────────┐     │
│     │  📧  Get in Touch                           │     │
│     │                                             │     │
│     │  We'd love to hear from you. Whether you    │     │
│     │  have questions, feedback, or ideas -       │     │
│     │  we're here to help.                        │     │
│     └─────────────────────────────────────────────┘     │
│                                                         │
│     ┌─────────────────────────────────────────────┐     │
│     │  First Name *          Last Name *          │     │
│     │  [____________]        [____________]       │     │
│     │                                             │     │
│     │  Email *                                    │     │
│     │  [____________________________]             │     │
│     │                                             │     │
│     │  Company (optional)                         │     │
│     │  [____________________________]             │     │
│     │                                             │     │
│     │  What brings you here? (optional)           │     │
│     │  [ ] Evaluating for training/workshops      │     │
│     │  [ ] Evaluating for teaching                │     │
│     │  [ ] Evaluating for consulting              │     │
│     │  [ ] General question                       │     │
│     │  [ ] Feature request                        │     │
│     │  [ ] Other                                  │     │
│     │                                             │     │
│     │  Your Message *                             │     │
│     │  [____________________________]             │     │
│     │  [____________________________]             │     │
│     │  [____________________________]             │     │
│     │                                             │     │
│     │  ┌─────────────────────────────────────┐    │     │
│     │  │  [Cloudflare Turnstile Widget]      │    │     │
│     │  └─────────────────────────────────────┘    │     │
│     │                                             │     │
│     │  By submitting this form, your data will    │     │
│     │  be used to process your request.           │     │
│     │  See our Privacy Policy.                    │     │
│     │                                             │     │
│     │        [ Send Message ]                     │     │
│     │                                             │     │
│     └─────────────────────────────────────────────┘     │
│                                                         │
├─────────────────────────────────────────────────────────┤
│  Footer (existing)                                      │
└─────────────────────────────────────────────────────────┘
```

---

## 4. Form Fields

| Field | Type | Required | Validation |
|-------|------|----------|------------|
| First Name | text | Yes | Min 2 chars, max 100 |
| Last Name | text | Yes | Min 2 chars, max 100 |
| Email | email | Yes | Valid email format |
| Company | text | No | Max 200 chars |
| Interests | checkbox group | No | Multi-select allowed |
| Message | textarea | Yes | Min 10 chars, max 2000 |

### Interest Options (Sophie's input)
Multi-select checkboxes:
- Evaluating for training/workshops
- Evaluating for teaching
- Evaluating for consulting
- General question
- Feature request
- Other

---

## 5. User Interactions

### Validation
- Inline validation on blur
- Error messages appear below each field in red
- Submit button disabled until Turnstile completed

### Success State
After successful submission, show:
```
┌─────────────────────────────────────────────┐
│  ✓  Message Sent!                           │
│                                             │
│  Thank you for reaching out. We'll get      │
│  back to you within 1-2 business days.      │
│                                             │
│  [Back to Home]                             │
└─────────────────────────────────────────────┘
```

### Error State
If submission fails:
```
┌─────────────────────────────────────────────┐
│  ⚠  Something went wrong                    │
│                                             │
│  We couldn't send your message. Please      │
│  try again or email us directly at          │
│  contact@masem.at                           │
│                                             │
│  [Try Again]                                │
└─────────────────────────────────────────────┘
```

---

## 6. Responsive Design

### Desktop (>768px)
- Two-column layout for First/Last name
- Max width: 600px centered

### Mobile (<768px)
- Single column layout
- Full width with padding

---

## 7. Accessibility

- All form fields have associated labels
- Error messages linked via aria-describedby
- Focus states clearly visible
- Keyboard navigation supported
- Color contrast meets WCAG AA

---

## 8. Legal Notice

Short notice below form (not a checkbox):
> "By submitting this form, your data will be used to process your request (legal basis: Art. 6(1)(b) GDPR). See our [Privacy Policy](/privacy)."

---

## 9. Files to Update

1. **Create:** `app/contact/page.tsx` - Contact form page
2. **Create:** `components/contact/ContactForm.tsx` - Form component
3. **Update:** `components/layout/Footer.tsx` - Change mailto to /contact link
4. **Update:** `app/whats-next/page.tsx` - Change mailto to /contact link
5. **Update:** `app/privacy/page.tsx` - Add contact form data section
