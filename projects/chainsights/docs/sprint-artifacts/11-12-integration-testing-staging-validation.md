# Story 11.12: Integration Testing & Staging Validation

Status: drafted

## Story

**As a** team
**I want** full integration testing on staging
**So that** we can confidently merge to production

## Acceptance Criteria

1. - [ ] Stripe test payment → donation recorded
2. - [ ] Coinbase test payment → donation recorded
3. - [ ] `/api/donations/total` returns correct sum
4. - [ ] `/impact` page displays correctly
5. - [ ] Confirmation page shows donation for both providers
6. - [ ] No console errors
7. - [ ] Mobile responsive

## Tasks / Subtasks

- [ ] Task 1: Deploy to staging
  - [ ] 1.1 Push all Epic 11 changes to develop branch
  - [ ] 1.2 Deploy develop to staging environment
  - [ ] 1.3 Verify all new endpoints are accessible

- [ ] Task 2: Stripe payment flow
  - [ ] 2.1 Navigate to checkout page
  - [ ] 2.2 Select Card payment method
  - [ ] 2.3 Complete Stripe test payment (card: 4242424242424242)
  - [ ] 2.4 Verify confirmation page shows donation message
  - [ ] 2.5 Verify donation record created in database
  - [ ] 2.6 Verify /api/donations/total includes the donation

- [ ] Task 3: Coinbase payment flow
  - [ ] 3.1 Navigate to checkout page
  - [ ] 3.2 Select Crypto payment method
  - [ ] 3.3 Complete Coinbase Commerce test charge
  - [ ] 3.4 Verify confirmation page shows donation message
  - [ ] 3.5 Verify donation record created in database
  - [ ] 3.6 Verify /api/donations/total includes the donation

- [ ] Task 4: Impact page testing
  - [ ] 4.1 Navigate to /impact
  - [ ] 4.2 If total < €500: verify "Coming Soon" view
  - [ ] 4.3 If total >= €500: verify donation total displayed
  - [ ] 4.4 Verify hoki.help link works
  - [ ] 4.5 Test mobile responsiveness

- [ ] Task 5: Cross-browser testing
  - [ ] 5.1 Chrome (desktop)
  - [ ] 5.2 Safari (desktop)
  - [ ] 5.3 Firefox (desktop)
  - [ ] 5.4 Chrome (mobile - Android)
  - [ ] 5.5 Safari (mobile - iOS)

- [ ] Task 6: Console & error checking
  - [ ] 6.1 Open DevTools on all pages
  - [ ] 6.2 Verify no JavaScript errors
  - [ ] 6.3 Verify no network errors (4xx/5xx)
  - [ ] 6.4 Verify no accessibility warnings

- [ ] Task 7: Final sign-off
  - [ ] 7.1 Document any issues found
  - [ ] 7.2 Fix critical issues if any
  - [ ] 7.3 Get Mario's approval to merge to production

## Dev Notes

### CRITICAL CONSTRAINT

> **ALL CODE CHANGES GO TO `develop` BRANCH ONLY!**

### Prerequisites

Before running this story, ensure all previous Epic 11 stories are complete:
- [x] Story 11-1: Database Schema for Donations
- [x] Story 11-2: Donation Queries Library
- [x] Story 11-3: Stripe Webhook - Add Donation Tracking
- [x] Story 11-4: Coinbase Commerce Integration
- [x] Story 11-5: Crypto Checkout API Endpoint
- [x] Story 11-6: Payment Method Selection Component
- [x] Story 11-7: Crypto Checkout Modal
- [x] Story 11-8: Confirmation Page - Donation Message
- [x] Story 11-9: Donations Total API Endpoint
- [x] Story 11-10: Impact Page
- [x] Story 11-11: Webhook Health Monitoring

### Test Credentials

**Stripe Test Mode:**
- Card: 4242 4242 4242 4242
- Expiry: Any future date
- CVC: Any 3 digits

**Coinbase Commerce:**
- Use sandbox/test mode
- Complete charge with test wallet

### API Endpoints to Verify

| Endpoint | Method | Expected |
|----------|--------|----------|
| `/api/donations/total` | GET | Returns `{ total, count, currency }` |
| `/api/crypto-checkout` | POST | Creates Coinbase charge |
| `/api/webhooks/stripe` | POST | Processes Stripe events |
| `/api/webhooks/coinbase` | POST | Processes Coinbase events |
| `/api/cron/webhook-health` | POST | Returns health status |

### Database Verification Query

```sql
-- Check donation records
SELECT * FROM donations ORDER BY created_at DESC LIMIT 10;

-- Verify total
SELECT SUM(donation_amount) as total, COUNT(*) as count
FROM donations
WHERE project = 'chainsights';
```

### Known Staging Environment Details

- URL: [staging.chainsights.one or similar]
- Environment: Vercel Preview/Staging
- Database: Neon staging branch

### Issue Tracking

| Issue | Severity | Status | Resolution |
|-------|----------|--------|------------|
| (none yet) | - | - | - |

## Dev Agent Record

### Context Reference

This is a manual testing/validation story that requires human execution.
Created by autonomous create-story workflow.

### Agent Model Used

Claude Opus 4.5

### Completion Notes List

(To be filled during testing)

### File List

No code changes - this is a testing/validation story.

## Senior Developer Review

(Completed after testing)

## Change Log

- 2026-01-21: Story created by autonomous workflow - ready for manual testing
