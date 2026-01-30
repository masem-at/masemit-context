# Story 4.2: Implement Opt-Out Submission and Email Confirmation

**Status:** done
**Epic:** 4 - Opt-Out Management System
**Story ID:** 4.2
**Created:** 2025-12-23
**Ultimate Context Engine Analysis:** Complete

---

## Story

**As a** DAO representative submitting an opt-out request,
**I want to** receive immediate confirmation that my request was received,
**So that** I know the process has started and my DAO will be removed within 24 hours.

---

## Acceptance Criteria

### Given-When-Then Format (from Epic 4)

**Given** I submit the opt-out form with valid data
**When** the form is processed
**Then** data is sent to API route `/api/rankings/opt-out`

**And** API route:
- Validates input using Zod schema from Story 3.3
- Creates record in `opt_out_requests` table (new table):
  - `id` (uuid, primary key)
  - `dao_name` (text, not null)
  - `contact_email` (text, not null)
  - `reason` (text, nullable)
  - `status` (enum: pending, processed)
  - `requested_at` (timestamp, default now)
  - `processed_at` (timestamp, nullable)
- Sends confirmation email via Resend:
  - Subject: "DAO Opt-Out Request Received - ChainSights"
  - Body: Confirms receipt, 24-hour removal guarantee, request ID
- Returns success response with request ID

**And** user sees success message: "Request received! Check your email for confirmation."

**And** if email fails, request is still stored but response includes warning

**And** audit log records the opt-out request (FR66)

---

## Critical Developer Context

### MISSION - IMPLEMENT API BACKEND FOR OPT-OUT FORM

This story implements the **backend processing** for the opt-out form created in Story 4.1. You are creating the API endpoint that receives form submissions, stores them in the database, and sends confirmation emails.

**What You're Building:**
1. **Database Schema** - Add `opt_out_requests` table to Drizzle schema
2. **API Route** - `POST /api/rankings/opt-out` endpoint
3. **Email Service** - `sendOptOutConfirmationEmail()` function
4. **Email Template** - Professional HTML email template

**What Already Exists (REUSE, DO NOT RECREATE):**
- ✅ **Validation Schemas:** `src/lib/validation/opt-out.ts` - Complete Zod schemas (Story 3.3)
- ✅ **Sanitization Utils:** `src/lib/validation/sanitize.ts` - XSS protection (Story 3.3)
- ✅ **Legal Constants:** `src/lib/constants/legal-disclaimers.ts` - LEGAL_CONTACT_EMAIL
- ✅ **Form Component:** `src/components/features/opt-out/OptOutForm.tsx` - Submits to this endpoint
- ✅ **Email Service Base:** `src/lib/email/index.ts` - Resend client setup
- ✅ **Database Connection:** `src/lib/db/index.ts` - Drizzle client

**What Story 4.3 Will Handle (NOT THIS STORY):**
- ❌ Admin dashboard to view/process requests
- ❌ Marking DAOs as opted-out in rankings
- ❌ Automated processing logic

---

### EXISTING CODE ANALYSIS

#### ✅ Validation Schemas (READY TO USE)

**File:** `src/lib/validation/opt-out.ts`

```typescript
import { validateOptOutForm, OptOutFormInput } from '@/lib/validation/opt-out'

// Server-side validation
const result = validateOptOutForm(body)
if (!result.success) {
  // result.error.issues contains field-level errors
}
// result.data is typed, sanitized, and safe to use
```

**Validation Rules Already Defined:**
- `daoName`: Required, 2-100 chars, alphanumeric + spaces/hyphens/underscores
- `contactEmail`: Required, valid email format, auto-lowercased and trimmed
- `reason`: Optional, max 500 chars, HTML-sanitized via `sanitizeText()`

---

#### ✅ Email Service (EXTEND THIS)

**File:** `src/lib/email/index.ts`

```typescript
import { Resend } from 'resend'

const resend = process.env.RESEND_API_KEY ? new Resend(process.env.RESEND_API_KEY) : null
const FROM_EMAIL = process.env.EMAIL_FROM || 'ChainSights <hello@chainsights.one>'

export function isEmailConfigured(): boolean {
  return !!resend
}

// ADD NEW FUNCTION HERE:
export async function sendOptOutConfirmationEmail(options: {
  to: string
  daoName: string
  requestId: string
}): Promise<void> {
  // Implementation
}
```

---

#### ⚠️ Database Schema (NEEDS NEW TABLE)

**File:** `src/lib/db/schema.ts`

**Add this new table:**

```typescript
import { pgTable, uuid, text, timestamp, pgEnum, index } from 'drizzle-orm/pg-core'

// Add enum for opt-out status
export const optOutStatusEnum = pgEnum('opt_out_status', ['pending', 'processed'])

// Add opt_out_requests table
export const optOutRequests = pgTable('opt_out_requests', {
  id: uuid('id').defaultRandom().primaryKey(),
  daoName: text('dao_name').notNull(),
  contactEmail: text('contact_email').notNull(),
  reason: text('reason'),
  status: optOutStatusEnum('status').default('pending').notNull(),
  requestedAt: timestamp('requested_at').defaultNow().notNull(),
  processedAt: timestamp('processed_at'),
  createdAt: timestamp('created_at').defaultNow().notNull(),
}, (table) => ({
  emailIdx: index('opt_out_requests_email_idx').on(table.contactEmail),
  statusIdx: index('opt_out_requests_status_idx').on(table.status),
}))

// Export types
export type OptOutRequest = typeof optOutRequests.$inferSelect
export type NewOptOutRequest = typeof optOutRequests.$inferInsert
```

---

#### ⚠️ API Route (CREATE NEW)

**File:** `src/app/api/rankings/opt-out/route.ts`

**Note:** This file may exist with different functionality. **REPLACE** with Epic 4 implementation.

---

### PREVIOUS STORY INTELLIGENCE

#### From Story 4.1 (Opt-Out Form)

**Key Learnings:**
- Zod 4.x uses `result.error.issues` (not `result.error.errors`)
- Form expects response format: `{ success: true }` or `{ error: { message: string } }`
- Form handles 404/405 gracefully with fallback email message
- Form resets and shows success state on 200 response

**Form Request Format:**
```typescript
const response = await fetch('/api/rankings/opt-out', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    daoName: string,      // Required, validated
    contactEmail: string, // Required, validated, lowercased
    reason?: string       // Optional, sanitized
  }),
})
```

**Expected Success Response:**
```json
{
  "success": true,
  "requestId": "uuid-string"
}
```

**Expected Error Response:**
```json
{
  "error": {
    "message": "User-friendly error message"
  }
}
```

---

### API ROUTE IMPLEMENTATION PATTERN

**File:** `src/app/api/rankings/opt-out/route.ts`

```typescript
import { NextRequest, NextResponse } from 'next/server'
import { db } from '@/lib/db'
import { optOutRequests } from '@/lib/db/schema'
import { validateOptOutForm } from '@/lib/validation/opt-out'
import { isEmailConfigured, sendOptOutConfirmationEmail } from '@/lib/email'

export async function POST(request: NextRequest) {
  try {
    // 1. Parse request body
    const body = await request.json()

    // 2. Validate with Zod (reuse Story 3.3 schemas)
    const result = validateOptOutForm(body)
    if (!result.success) {
      return NextResponse.json(
        { error: { message: 'Validation failed', details: result.error.issues } },
        { status: 400 }
      )
    }

    const { daoName, contactEmail, reason } = result.data

    // 3. Check if database is configured
    if (!db) {
      return NextResponse.json(
        { error: { message: 'Database not configured' } },
        { status: 503 }
      )
    }

    // 4. Insert into database
    const [optOutRequest] = await db
      .insert(optOutRequests)
      .values({ daoName, contactEmail, reason })
      .returning()

    // 5. Send confirmation email (graceful failure)
    let emailWarning: string | undefined
    if (isEmailConfigured()) {
      try {
        await sendOptOutConfirmationEmail({
          to: contactEmail,
          daoName,
          requestId: optOutRequest.id,
        })
      } catch (emailError) {
        console.error('Failed to send confirmation email:', emailError)
        emailWarning = 'Request saved but email delivery failed. Check spam or contact support.'
      }
    } else {
      emailWarning = 'Email service not configured. Request saved successfully.'
    }

    // 6. Return success response
    return NextResponse.json({
      success: true,
      requestId: optOutRequest.id,
      ...(emailWarning && { warning: emailWarning }),
    })

  } catch (error) {
    console.error('Opt-out submission error:', error)
    return NextResponse.json(
      { error: { message: 'Failed to process request. Please try again.' } },
      { status: 500 }
    )
  }
}
```

---

### EMAIL TEMPLATE REQUIREMENTS

**Subject:** `DAO Opt-Out Request Received - ChainSights`

**Content Must Include:**
1. Confirmation that request was received
2. DAO name and request ID
3. 24-hour removal guarantee
4. What happens next (removal from public rankings, data retention)
5. Re-inclusion instructions (email hello@chainsights.one)
6. Contact information for questions

**HTML Template Pattern:**
```typescript
function getOptOutConfirmationHtml(daoName: string, requestId: string): string {
  return `
    <!DOCTYPE html>
    <html>
    <head>
      <meta charset="utf-8">
      <meta name="viewport" content="width=device-width, initial-scale=1">
    </head>
    <body style="font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif; background-color: #f4f4f5; padding: 20px;">
      <div style="max-width: 600px; margin: 0 auto; background: white; border-radius: 8px; padding: 32px;">
        <h1 style="color: #0F2C55; margin-bottom: 24px;">Opt-Out Request Received</h1>

        <p style="color: #374151; line-height: 1.6;">
          We've received your request to remove <strong>${daoName}</strong> from the ChainSights public governance rankings.
        </p>

        <div style="background: #f0fdf9; border: 1px solid #00E5C0; border-radius: 8px; padding: 16px; margin: 24px 0;">
          <p style="margin: 0; color: #0F2C55;"><strong>Request ID:</strong> ${requestId}</p>
        </div>

        <h2 style="color: #0F2C55; font-size: 18px;">What Happens Next</h2>
        <ul style="color: #374151; line-height: 1.8;">
          <li>Your DAO will be removed from public rankings <strong>within 24 hours</strong></li>
          <li>Historical governance data will be retained internally but not displayed publicly</li>
          <li>Future GVS calculations will automatically skip your DAO</li>
        </ul>

        <p style="color: #6b7280; font-size: 14px; margin-top: 32px;">
          Want to be re-included later? Email us at
          <a href="mailto:hello@chainsights.one" style="color: #00E5C0;">hello@chainsights.one</a>
        </p>

        <hr style="border: none; border-top: 1px solid #e5e7eb; margin: 32px 0;">

        <p style="color: #9ca3af; font-size: 12px; text-align: center;">
          ChainSights | Transparent DAO Governance Analytics<br>
          <a href="https://chainsights.one" style="color: #00E5C0;">chainsights.one</a>
        </p>
      </div>
    </body>
    </html>
  `
}
```

---

## Implementation Strategy

### Phase 1: Database Schema (15 min)

1. Open `src/lib/db/schema.ts`
2. Add `optOutStatusEnum` enum definition
3. Add `optOutRequests` table definition
4. Add type exports
5. Run `npm run db:generate` to generate migration
6. Run `npm run db:push` to apply schema

### Phase 2: Email Function (30 min)

1. Open `src/lib/email/index.ts`
2. Add `sendOptOutConfirmationEmail()` function
3. Create HTML and plain text templates
4. Handle Resend API errors gracefully

### Phase 3: API Route (45 min)

1. Create/replace `src/app/api/rankings/opt-out/route.ts`
2. Implement POST handler with:
   - Request parsing
   - Zod validation (reuse existing schemas)
   - Database insertion
   - Email sending with graceful failure
   - Proper error responses

### Phase 4: Testing (30 min)

1. Run `npm run build` to verify TypeScript compiles
2. Test with form submission on `/rankings/opt-out`
3. Verify database record created
4. Verify email received (or warning if not configured)
5. Test error scenarios

---

## Tasks/Subtasks

### Task 1: Create Database Schema
- [x] Add `optOutStatusEnum` to `src/lib/db/schema.ts`
- [x] Add `optOutRequests` table with all required fields
- [x] Add indexes for email and status
- [x] Export `OptOutRequest` and `NewOptOutRequest` types
- [x] Run `npm run db:generate` to create migration
- [x] Run `npm run db:push` to apply to database

### Task 2: Create Email Function
- [x] Add `sendOptOutConfirmationEmail()` to `src/lib/email/index.ts`
- [x] Create HTML email template with brand styling
- [x] Create plain text fallback
- [x] Include: request ID, DAO name, 24-hour guarantee, next steps
- [x] Handle Resend API errors

### Task 3: Implement API Route
- [x] Create `src/app/api/rankings/opt-out/route.ts`
- [x] Import validation from `@/lib/validation/opt-out`
- [x] Parse and validate request body with Zod
- [x] Insert record into `opt_out_requests` table
- [x] Call `sendOptOutConfirmationEmail()` with graceful failure
- [x] Return success response with request ID
- [x] Return proper error responses (400, 500, 503)

### Task 4: Error Handling
- [x] Handle validation errors (400)
- [x] Handle database errors (500)
- [x] Handle email failures gracefully (store record, return warning)
- [x] Handle missing database config (503)
- [x] Log errors to console (not sensitive data)

### Task 5: Testing
- [x] Run `npm run build` - verify compilation
- [x] Test form submission with valid data
- [x] Test form submission with invalid data
- [x] Verify database record created correctly
- [x] Verify email sent (if RESEND_API_KEY configured)
- [x] Test graceful degradation when email fails

### Task 6: Update Story Status
- [x] Mark all tasks complete
- [x] Update status to "done"
- [x] Update sprint-status.yaml
- [x] Commit with message: `feat(story-4.2): implement opt-out submission and email`

---

## Technical Requirements

### File Structure

```
src/
├── app/
│   └── api/
│       └── rankings/
│           └── opt-out/
│               └── route.ts        # API route (CREATE/REPLACE)
├── lib/
│   ├── db/
│   │   └── schema.ts               # Add opt_out_requests table
│   ├── email/
│   │   └── index.ts                # Add sendOptOutConfirmationEmail()
│   └── validation/
│       └── opt-out.ts              # ALREADY EXISTS - reuse
```

### Environment Variables Required

```bash
# Email (Resend) - REQUIRED for email sending
RESEND_API_KEY=re_...
EMAIL_FROM=ChainSights <hello@chainsights.one>

# Database - ALREADY CONFIGURED
DATABASE_URL=postgresql://...
```

### API Response Formats

**Success (200):**
```json
{
  "success": true,
  "requestId": "550e8400-e29b-41d4-a716-446655440000"
}
```

**Success with Warning (200):**
```json
{
  "success": true,
  "requestId": "550e8400-e29b-41d4-a716-446655440000",
  "warning": "Request saved but email delivery failed."
}
```

**Validation Error (400):**
```json
{
  "error": {
    "message": "Validation failed",
    "details": [
      { "path": ["contactEmail"], "message": "Invalid email format" }
    ]
  }
}
```

**Server Error (500):**
```json
{
  "error": {
    "message": "Failed to process request. Please try again."
  }
}
```

**Service Unavailable (503):**
```json
{
  "error": {
    "message": "Database not configured"
  }
}
```

---

## Definition of Done

- [x] `opt_out_requests` table created in database schema
- [x] Migration generated and applied
- [x] `sendOptOutConfirmationEmail()` function created
- [x] Email template includes all required content (request ID, 24-hour guarantee, next steps)
- [x] API route handles POST requests correctly
- [x] Validation uses existing Zod schemas from Story 3.3
- [x] Database record created on successful submission
- [x] Email sent on successful submission
- [x] Email failure handled gracefully (record still saved)
- [x] Proper error responses for all failure scenarios
- [x] TypeScript compiles without errors
- [x] Form submission works end-to-end
- [x] Build succeeds (`npm run build`)
- [x] Unit tests pass (12 tests)

---

## Dev Agent Record

### Context Reference
<!-- Ultimate Context Engine analysis completed -->

### Agent Model Used
Claude Opus 4.5

### Debug Log References
- None

### Completion Notes List
- 2025-12-23: Story 4.2 created with Ultimate Context Engine analysis
- Builds on Story 4.1 (form component) - API backend implementation
- Reuse validation schemas from Story 3.3 - DO NOT recreate
- Email service already configured - just add new function
- Database schema needs new table - follow Drizzle patterns
- Graceful email failure is REQUIRED - never block opt-out due to email

### File List

**Files Modified:**
- `src/app/api/rankings/opt-out/route.ts` - API route handler (replaced existing)
- `src/lib/db/schema.ts` - Added optOutRequests table and optOutStatusEnum
- `src/lib/email/index.ts` - Added sendOptOutConfirmationEmail() with HTML/text templates

**Files Created:**
- `drizzle/0005_awesome_lizard.sql` - Database migration
- `drizzle/meta/0005_snapshot.json` - Migration metadata
- `tests/unit/api/rankings/opt-out.test.ts` - Unit tests (12 tests)

**Files Reused (NOT modified):**
- `src/lib/validation/opt-out.ts` - Zod schemas
- `src/lib/validation/sanitize.ts` - Sanitization utils
- `src/lib/constants/legal-disclaimers.ts` - Legal text
- `src/components/features/opt-out/OptOutForm.tsx` - Form component

---

## References

**Epic 4 Definition:** `docs/epics.md` - Opt-Out Management System
**Architecture:** `docs/architecture.md` - API patterns, email service, database
**Project Context:** `docs/project_context.md` - Coding standards
**Story 4.1:** `docs/sprint-artifacts/4-1-create-opt-out-form-page.md` - Form implementation
**Validation Schemas:** `src/lib/validation/opt-out.ts` (Story 3.3)
**Email Service:** `src/lib/email/index.ts`
**Database Schema:** `src/lib/db/schema.ts`

---

## Change Log

- **2025-12-23**: Story 4.2 created with Ultimate Context Engine analysis - comprehensive API backend implementation guide
- **2025-12-23**: Implementation complete - database schema, email function, API route, 12 unit tests
- **2025-12-23**: Code review fixes - Added requestId to email template (H1), fixed email subject (M1), added FR66 audit logging (M3), HTML-escaped daoName (L2)

---

## Senior Developer Review (AI)

**Review Date:** 2025-12-23
**Reviewer:** Claude Opus 4.5

### Issues Found & Fixed

| Severity | Issue | Resolution |
|----------|-------|------------|
| HIGH | Email template missing Request ID per AC | Added requestId parameter and display in both HTML/text templates |
| MEDIUM | Email subject differed from AC spec | Changed to "DAO Opt-Out Request Received - ChainSights" |
| MEDIUM | Story file not updated with implementation | Updated all tasks, status, File List |
| MEDIUM | Missing audit log per FR66 | Added `[AUDIT]` console.log with structured data |
| MEDIUM | Test file not in story File List | Added to File List section |
| LOW | emailError variable unused | Now included in emailWarning response |
| LOW | No HTML escaping in email template | Added escapeHtml() function for defense-in-depth |

### Verification
- ✅ All 12 unit tests pass
- ✅ Build succeeds
- ✅ All ACs implemented
- ✅ All tasks marked complete

**Outcome:** APPROVED - Story ready for done status

---
