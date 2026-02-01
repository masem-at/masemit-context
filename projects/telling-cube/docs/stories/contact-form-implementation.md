# Story: Contact Form Implementation

**Status:** Ready for Development
**Assigned:** James (Dev)
**Priority:** Medium
**Effort:** 1.5 - 2 days

---

## Summary

Implement a contact form to replace mailto links, with Cloudflare Turnstile spam protection and database storage for inquiry tracking.

---

## References

- **UX Spec:** `docs/ux/contact-form-spec.md`
- **Technical Spec:** `docs/architecture/contact-form-technical-spec.md`
- **GDPR Guide:** `docs/_masemIT/ideas/contact-form.md`

---

## Acceptance Criteria

### Core Form
- [ ] Contact form page at `/contact`
- [ ] Fields: First Name*, Last Name*, Email*, Company, Message*, Interests (checkboxes)
- [ ] All required fields validated (client + server)
- [ ] Inline validation on blur with error messages
- [ ] Responsive layout (2-col desktop, 1-col mobile)

### Interest Checkboxes
- [ ] Multi-select checkboxes with options:
  - Evaluating for training/workshops
  - Evaluating for teaching
  - Evaluating for consulting
  - General question
  - Feature request
  - Other

### Spam Protection
- [ ] Cloudflare Turnstile widget integrated
- [ ] Server-side token verification
- [ ] Submit button disabled until Turnstile completed
- [ ] Rate limiting: 5 submissions/minute per IP

### Database
- [ ] `ContactSubmission` model added to Prisma schema
- [ ] Migration created and applied
- [ ] Submissions stored with all form data + IP + timestamp

### API Endpoint
- [ ] `POST /api/contact` endpoint
- [ ] Zod validation schema
- [ ] Input sanitization (strip HTML)
- [ ] Proper error responses with field-level details

### UX States
- [ ] Success state: "Message Sent!" with back-to-home link
- [ ] Error state: Friendly message with fallback email (contact@masem.at)
- [ ] Loading state: Spinner/disabled during submission

### Navigation Updates
- [ ] Footer: Change "Contact Support" mailto → `/contact` link
- [ ] What's Next page: Change "Email Us Your Ideas" mailto → `/contact` link

### Privacy Policy Update
- [ ] Add "Contact Form Data" to "Data We Collect" section
- [ ] Add retention period: "Contact form submissions retained for 12 months"
- [ ] Add Cloudflare to "Third-Party Services" section

### Legal Notice
- [ ] Display below form: "By submitting this form, your data will be used to process your request. See our [Privacy Policy](/privacy)."

---

## Environment Variables Required

Add to `.env` (Mario to provide values):
```
NEXT_PUBLIC_TURNSTILE_SITE_KEY=xxx
TURNSTILE_SECRET_KEY=xxx
```

---

## Dependencies

```bash
pnpm add @marsidev/react-turnstile
```

---

## Files to Create

1. `app/contact/page.tsx` - Contact page
2. `app/api/contact/route.ts` - API endpoint
3. `components/contact/ContactForm.tsx` - Form component
4. `lib/validations/contact.ts` - Zod schema

---

## Files to Modify

1. `prisma/schema.prisma` - Add ContactSubmission model
2. `components/layout/Footer.tsx` - Update contact link
3. `app/whats-next/page.tsx` - Update CTA button
4. `app/privacy/page.tsx` - Add contact form data section
5. `.env.example` - Add Turnstile env vars

---

## Out of Scope

- Email notifications to support (can be added later)
- Admin panel to view submissions (query DB directly for now)
- "How did you find us?" tracking (later phase)
- Newsletter signup (no newsletter yet)

---

## Testing Notes

- Test all validation rules (empty, too short, too long, invalid email)
- Test Turnstile in development mode (Cloudflare provides test keys)
- Test rate limiting (submit 6 times quickly)
- Test mobile responsive layout
- Verify submission appears in database
