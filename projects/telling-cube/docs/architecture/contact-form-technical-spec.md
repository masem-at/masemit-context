# Contact Form - Technical Specification

**Author:** Winston (Architecture)
**Date:** December 2024
**Status:** Ready for Review

---

## 1. Overview

Technical implementation for the contact form feature, including database schema, API endpoint, and Cloudflare Turnstile integration.

---

## 2. Database Schema

### New Model: ContactSubmission

Add to `prisma/schema.prisma`:

```prisma
// Contact Form Submissions
model ContactSubmission {
  id              String    @id @default(uuid())

  // Contact Information
  firstName       String    @db.VarChar(100)
  lastName        String    @db.VarChar(100)
  email           String    @db.VarChar(255)
  company         String?   @db.VarChar(200)

  // Message
  message         String    @db.Text

  // Interests (multi-select options)
  interests       String[]  @default([])

  // Security & Metadata
  ipAddress       String?   @db.VarChar(45)
  userAgent       String?
  turnstileValid  Boolean   @default(false)

  // Status Tracking
  status          String    @default("new")  // 'new' | 'read' | 'responded' | 'archived'

  // Timestamps
  createdAt       DateTime  @default(now())

  // Indexes
  @@index([status])
  @@index([createdAt])
  @@index([email])

  @@map("contact_submissions")
}
```

### Interest Options (Enum-like values)
Store as string array. Valid values:
- `training` - Evaluating for training/workshops
- `teaching` - Evaluating for teaching
- `consulting` - Evaluating for consulting
- `question` - General question
- `feature` - Feature request
- `other` - Other

---

## 3. API Endpoint

### POST /api/contact

**File:** `app/api/contact/route.ts`

#### Request Body
```typescript
interface ContactRequest {
  firstName: string;      // required, 2-100 chars
  lastName: string;       // required, 2-100 chars
  email: string;          // required, valid email
  company?: string;       // optional, max 200 chars
  message: string;        // required, 10-2000 chars
  interests?: string[];   // optional, array of valid interest codes
  turnstileToken: string; // required, Cloudflare Turnstile token
}
```

#### Response
```typescript
// Success (201)
interface ContactResponse {
  success: true;
  message: string;
}

// Error (400 | 429 | 500)
interface ContactErrorResponse {
  success: false;
  error: string;
  details?: Record<string, string>; // Field-level errors
}
```

#### Implementation Steps
1. Validate Turnstile token with Cloudflare API
2. Validate request body (zod schema)
3. Rate limit check (by IP)
4. Sanitize inputs (strip HTML/JS)
5. Store in database
6. (Optional) Send notification email to support
7. Return success response

---

## 4. Cloudflare Turnstile Integration

### Environment Variables
Add to `.env`:
```
TURNSTILE_SITE_KEY=xxx        # Public key for frontend widget
TURNSTILE_SECRET_KEY=xxx      # Secret key for server verification
```

### Server-Side Verification
```typescript
async function verifyTurnstile(token: string, ip?: string): Promise<boolean> {
  const response = await fetch(
    'https://challenges.cloudflare.com/turnstile/v0/siteverify',
    {
      method: 'POST',
      headers: { 'Content-Type': 'application/x-www-form-urlencoded' },
      body: new URLSearchParams({
        secret: process.env.TURNSTILE_SECRET_KEY!,
        response: token,
        ...(ip && { remoteip: ip }),
      }),
    }
  );
  const data = await response.json();
  return data.success === true;
}
```

### Frontend Widget
Use `@marsidev/react-turnstile` package:
```tsx
import { Turnstile } from '@marsidev/react-turnstile';

<Turnstile
  siteKey={process.env.NEXT_PUBLIC_TURNSTILE_SITE_KEY!}
  onSuccess={(token) => setTurnstileToken(token)}
/>
```

---

## 5. Rate Limiting

Simple IP-based rate limiting using in-memory store (sufficient for PoC):

```typescript
const rateLimitMap = new Map<string, { count: number; resetAt: number }>();

function checkRateLimit(ip: string): boolean {
  const now = Date.now();
  const limit = rateLimitMap.get(ip);

  if (!limit || now > limit.resetAt) {
    rateLimitMap.set(ip, { count: 1, resetAt: now + 60000 }); // 1 minute window
    return true;
  }

  if (limit.count >= 5) { // Max 5 submissions per minute
    return false;
  }

  limit.count++;
  return true;
}
```

---

## 6. Input Validation (Zod Schema)

```typescript
import { z } from 'zod';

const validInterests = ['training', 'teaching', 'consulting', 'question', 'feature', 'other'];

export const contactFormSchema = z.object({
  firstName: z.string().min(2).max(100).trim(),
  lastName: z.string().min(2).max(100).trim(),
  email: z.string().email().max(255).toLowerCase(),
  company: z.string().max(200).trim().optional(),
  message: z.string().min(10).max(2000).trim(),
  interests: z.array(z.enum(validInterests as [string, ...string[]])).optional(),
  turnstileToken: z.string().min(1),
});
```

---

## 7. Security Measures

1. **Turnstile** - Bot protection
2. **Rate Limiting** - 5 requests/minute per IP
3. **Input Validation** - Zod schema validation
4. **Input Sanitization** - Strip HTML tags, prevent XSS
5. **HTTPS** - Enforced by Vercel
6. **No SQL Injection** - Prisma parameterized queries

---

## 8. Dependencies to Add

```bash
pnpm add @marsidev/react-turnstile
```

No other new dependencies needed (zod already installed).

---

## 9. Files to Create/Modify

### Create
- `app/api/contact/route.ts` - API endpoint
- `app/contact/page.tsx` - Contact page
- `components/contact/ContactForm.tsx` - Form component
- `lib/validations/contact.ts` - Zod schema

### Modify
- `prisma/schema.prisma` - Add ContactSubmission model
- `components/layout/Footer.tsx` - Update link
- `app/whats-next/page.tsx` - Update CTA link
- `app/privacy/page.tsx` - Add contact form data section
- `.env.example` - Add Turnstile env vars

---

## 10. Migration

```bash
npx prisma migrate dev --name add_contact_submissions
```
