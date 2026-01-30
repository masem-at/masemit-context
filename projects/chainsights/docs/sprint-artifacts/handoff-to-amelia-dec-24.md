# Handoff to Amelia - Post-Launch Features (Dec 24, 2025)

**From:** Mary (Architect)
**To:** Amelia (Developer)
**Date:** December 24, 2025
**Context:** ChainSights Rankings already LAUNCHED. Continuing post-launch feature implementation.

---

## Feature 1: Discord/Forum Links ✅ COMPLETED

**Status:** Implementation complete, script ready to run

**What's Done:**
- ✅ Database schema updated (3 new fields: discordUrl, forumUrl, twitterUrl)
- ✅ Migration applied to production database
- ✅ UI components updated (ExpandableDAORow displays community links)
- ✅ UTM tracking implemented (?utm_source=chainsights appended automatically)
- ✅ Admin interface updated (Featured DAOs can be edited)
- ✅ TypeScript interfaces updated (LeaderboardRanking, FeaturedDAO)
- ✅ All 9 DAO community links researched

**Final Step for Feature 1:**
Run the population script to add links for the 9 DAOs:

```bash
npx tsx scripts/add-community-links.ts
```

**IMPORTANT:** The DATABASE_URL in `.env.local` is split across two lines. Fix it first:

**Current (broken):**
```
DATABASE_URL=postgresql://neondb_owner:npg_4vAy2DtbrgdQ@ep-muddy-cherry-agi4uxec-pooler.c-2.eu-central-1.aws.neon.tech/
neondb?sslmode=require&channel_binding=require
```

**Fixed (one line):**
```
DATABASE_URL=postgresql://neondb_owner:npg_4vAy2DtbrgdQ@ep-muddy-cherry-agi4uxec-pooler.c-2.eu-central-1.aws.neon.tech/neondb?sslmode=require&channel_binding=require
```

After fixing, run the script and verify links appear on `/rankings`.

---

## Feature 2: Privacy Policy Update (GDPR) ⏳ PENDING

**Priority:** P0 (MUST deploy BEFORE Feature 3)
**Timeline:** 30 minutes
**Owner:** Daniel (Legal) drafting, Amelia deploying

**Requirements:**
Add "Email Notifications" section to `/privacy` page documenting:
- Email collection purpose (rankings updates)
- Legal basis (consent, GDPR Article 6(1)(a))
- Data retention (until unsubscribe)
- User rights (access, deletion, unsubscribe)
- Processor details (Resend, Ireland)

**File to Edit:**
`src/app/privacy/page.tsx`

**Content Template:**
```markdown
## Email Notifications

We collect email addresses from users who opt-in to receive notifications about new DAO rankings.

**Legal Basis:** Consent (GDPR Article 6(1)(a))

**Purpose:** Sending weekly email notifications when new DAOs are analyzed and added to our rankings.

**Data Stored:** Email address, subscription timestamp, confirmation status, unsubscribe status

**Retention:** Email addresses are retained until you unsubscribe. You can unsubscribe at any time using the link provided in every email.

**Email Processor:** We use Resend (Ireland) as our email service provider. View their privacy policy at https://resend.com/legal/privacy-policy

**Your Rights:**
- Access your data via the privacy request form
- Delete your data (unsubscribe link in emails or privacy request form)
- Export your data via privacy request form
```

**Acceptance Criteria:**
- [ ] Privacy policy includes "Email Notifications" section
- [ ] Mentions Resend as email processor
- [ ] Documents user's right to unsubscribe at any time
- [ ] Links to Resend's privacy policy
- [ ] Deployed to production BEFORE email capture form

---

## Feature 3: Email Notification System ("Rankings Watch") ⏳ PENDING

**Priority:** P1
**Timeline:** 4-6 hours
**Dependencies:** Feature 2 MUST be deployed first

**User Value:** Users get notified when new DAOs are analyzed. Retention mechanism.

### Implementation Plan

#### Step 1: Database Migration
Create new table for email subscribers:

```sql
CREATE TABLE email_subscribers (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email TEXT NOT NULL UNIQUE,
  subscribed_at TIMESTAMP NOT NULL DEFAULT NOW(),
  confirmed BOOLEAN NOT NULL DEFAULT FALSE,
  confirmation_token TEXT UNIQUE,
  confirmation_sent_at TIMESTAMP,
  unsubscribe_token TEXT UNIQUE NOT NULL,
  unsubscribed BOOLEAN NOT NULL DEFAULT FALSE,
  unsubscribed_at TIMESTAMP,
  created_at TIMESTAMP NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMP NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_email_subscribers_email ON email_subscribers(email);
CREATE INDEX idx_email_subscribers_confirmed ON email_subscribers(confirmed);
CREATE INDEX idx_email_subscribers_unsubscribed ON email_subscribers(unsubscribed);
```

Update `src/lib/db/schema.ts`:
```typescript
export const emailSubscribers = pgTable('email_subscribers', {
  id: uuid('id').defaultRandom().primaryKey(),
  email: text('email').notNull().unique(),
  subscribedAt: timestamp('subscribed_at').defaultNow().notNull(),
  confirmed: boolean('confirmed').default(false).notNull(),
  confirmationToken: text('confirmation_token').unique(),
  confirmationSentAt: timestamp('confirmation_sent_at'),
  unsubscribeToken: text('unsubscribe_token').notNull().unique(),
  unsubscribed: boolean('unsubscribed').default(false).notNull(),
  unsubscribedAt: timestamp('unsubscribed_at'),
  createdAt: timestamp('created_at').defaultNow().notNull(),
  updatedAt: timestamp('updated_at').defaultNow().notNull(),
}, (table) => ({
  emailIdx: index('idx_email_subscribers_email').on(table.email),
  confirmedIdx: index('idx_email_subscribers_confirmed').on(table.confirmed),
  unsubscribedIdx: index('idx_email_subscribers_unsubscribed').on(table.unsubscribed),
}))
```

#### Step 2: Signup API Endpoint
Create `/api/subscribe` (POST):

```typescript
// src/app/api/subscribe/route.ts
import { NextRequest, NextResponse } from 'next/server'
import { z } from 'zod'
import { getDb } from '@/lib/db'
import { emailSubscribers } from '@/lib/db/schema'
import { eq } from 'drizzle-orm'
import { randomBytes } from 'crypto'
import { sendConfirmationEmail } from '@/lib/email/resend'

const subscribeSchema = z.object({
  email: z.string().email(),
})

export async function POST(request: NextRequest): Promise<NextResponse> {
  try {
    const body = await request.json()
    const { email } = subscribeSchema.parse(body)

    const db = getDb()

    // Check if already subscribed
    const [existing] = await db
      .select()
      .from(emailSubscribers)
      .where(eq(emailSubscribers.email, email))
      .limit(1)

    if (existing) {
      if (existing.confirmed) {
        return NextResponse.json({
          success: false,
          error: 'Already subscribed',
        }, { status: 400 })
      }

      // Resend confirmation
      await sendConfirmationEmail(email, existing.confirmationToken!)
      return NextResponse.json({
        success: true,
        message: 'Confirmation email resent',
      })
    }

    // Create new subscription
    const confirmationToken = randomBytes(32).toString('hex')
    const unsubscribeToken = randomBytes(32).toString('hex')

    await db.insert(emailSubscribers).values({
      email,
      confirmationToken,
      unsubscribeToken,
      confirmationSentAt: new Date(),
    })

    // Send confirmation email
    await sendConfirmationEmail(email, confirmationToken)

    return NextResponse.json({
      success: true,
      message: 'Check your email to confirm subscription',
    })
  } catch (error) {
    console.error('[Subscribe] Error:', error)
    return NextResponse.json({
      success: false,
      error: 'Failed to subscribe',
    }, { status: 500 })
  }
}
```

#### Step 3: Confirmation API Endpoint
Create `/api/subscribe/confirm` (GET):

```typescript
// src/app/api/subscribe/confirm/route.ts
import { NextRequest, NextResponse } from 'next/server'
import { getDb } from '@/lib/db'
import { emailSubscribers } from '@/lib/db/schema'
import { eq } from 'drizzle-orm'

export async function GET(request: NextRequest): Promise<NextResponse> {
  const token = request.nextUrl.searchParams.get('token')

  if (!token) {
    return NextResponse.redirect(new URL('/rankings?error=invalid_token', request.url))
  }

  const db = getDb()

  try {
    const [subscriber] = await db
      .select()
      .from(emailSubscribers)
      .where(eq(emailSubscribers.confirmationToken, token))
      .limit(1)

    if (!subscriber) {
      return NextResponse.redirect(new URL('/rankings?error=invalid_token', request.url))
    }

    if (subscriber.confirmed) {
      return NextResponse.redirect(new URL('/rankings?message=already_confirmed', request.url))
    }

    // Confirm subscription
    await db
      .update(emailSubscribers)
      .set({
        confirmed: true,
        updatedAt: new Date(),
      })
      .where(eq(emailSubscribers.id, subscriber.id))

    return NextResponse.redirect(new URL('/rankings?message=confirmed', request.url))
  } catch (error) {
    console.error('[Confirm] Error:', error)
    return NextResponse.redirect(new URL('/rankings?error=server_error', request.url))
  }
}
```

#### Step 4: Unsubscribe API Endpoint
Create `/api/subscribe/unsubscribe` (GET):

```typescript
// src/app/api/subscribe/unsubscribe/route.ts
import { NextRequest, NextResponse } from 'next/server'
import { getDb } from '@/lib/db'
import { emailSubscribers } from '@/lib/db/schema'
import { eq } from 'drizzle-orm'

export async function GET(request: NextRequest): Promise<NextResponse> {
  const token = request.nextUrl.searchParams.get('token')

  if (!token) {
    return NextResponse.redirect(new URL('/rankings?error=invalid_token', request.url))
  }

  const db = getDb()

  try {
    const [subscriber] = await db
      .select()
      .from(emailSubscribers)
      .where(eq(emailSubscribers.unsubscribeToken, token))
      .limit(1)

    if (!subscriber) {
      return NextResponse.redirect(new URL('/rankings?error=invalid_token', request.url))
    }

    if (subscriber.unsubscribed) {
      return NextResponse.redirect(new URL('/rankings?message=already_unsubscribed', request.url))
    }

    // Unsubscribe
    await db
      .update(emailSubscribers)
      .set({
        unsubscribed: true,
        unsubscribedAt: new Date(),
        updatedAt: new Date(),
      })
      .where(eq(emailSubscribers.id, subscriber.id))

    return NextResponse.redirect(new URL('/rankings?message=unsubscribed', request.url))
  } catch (error) {
    console.error('[Unsubscribe] Error:', error)
    return NextResponse.redirect(new URL('/rankings?error=server_error', request.url))
  }
}
```

#### Step 5: Resend Email Integration
Create email sending module:

```typescript
// src/lib/email/resend.ts
import { Resend } from 'resend'

const resend = new Resend(process.env.RESEND_API_KEY)

export async function sendConfirmationEmail(email: string, token: string) {
  const confirmUrl = `${process.env.NEXT_PUBLIC_BASE_URL}/api/subscribe/confirm?token=${token}`

  await resend.emails.send({
    from: 'ChainSights <notifications@chainsights.one>',
    to: email,
    subject: 'Confirm your Rankings Watch subscription',
    html: `
      <h2>Confirm Your Subscription</h2>
      <p>Thanks for subscribing to ChainSights Rankings Watch!</p>
      <p>Click the link below to confirm your email address:</p>
      <p><a href="${confirmUrl}">Confirm Subscription</a></p>
      <p>If you didn't request this, you can safely ignore this email.</p>
    `,
  })
}

export async function sendRankingsUpdateEmail(
  email: string,
  unsubscribeToken: string,
  newDAOs: Array<{ name: string; score: number; status: string }>
) {
  const unsubscribeUrl = `${process.env.NEXT_PUBLIC_BASE_URL}/api/subscribe/unsubscribe?token=${unsubscribeToken}`

  await resend.emails.send({
    from: 'ChainSights <notifications@chainsights.one>',
    to: email,
    subject: `${newDAOs.length} New DAOs Analyzed - Rankings Watch`,
    html: `
      <h2>New DAOs on ChainSights Rankings</h2>
      <p>We just analyzed ${newDAOs.length} new DAO${newDAOs.length > 1 ? 's' : ''} this week:</p>
      <ul>
        ${newDAOs.map(dao => `
          <li><strong>${dao.name}</strong> - ${dao.score}/10 (${dao.status})</li>
        `).join('')}
      </ul>
      <p><a href="${process.env.NEXT_PUBLIC_BASE_URL}/rankings">View Full Rankings</a></p>
      <hr>
      <p style="font-size: 12px; color: #666;">
        <a href="${unsubscribeUrl}">Unsubscribe</a> from Rankings Watch
      </p>
    `,
  })
}
```

#### Step 6: UI Component (Email Capture Form)
Add to `/rankings` page:

```typescript
// src/components/rankings-watch-signup.tsx
'use client'

import { useState } from 'react'
import { Button } from '@/components/ui/button'
import { Input } from '@/components/ui/input'

export function RankingsWatchSignup() {
  const [email, setEmail] = useState('')
  const [isSubmitting, setIsSubmitting] = useState(false)
  const [result, setResult] = useState<{
    type: 'success' | 'error'
    message: string
  } | null>(null)

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault()
    setIsSubmitting(true)
    setResult(null)

    try {
      const response = await fetch('/api/subscribe', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ email }),
      })

      const data = await response.json()

      if (data.success) {
        setResult({
          type: 'success',
          message: 'Check your email to confirm subscription!',
        })
        setEmail('')
      } else {
        setResult({
          type: 'error',
          message: data.error || 'Failed to subscribe',
        })
      }
    } catch (error) {
      setResult({
        type: 'error',
        message: 'Something went wrong. Please try again.',
      })
    } finally {
      setIsSubmitting(false)
    }
  }

  return (
    <div className="bg-gray-900 border border-gray-800 rounded-lg p-6">
      <h3 className="text-xl font-bold mb-2">Rankings Watch 📬</h3>
      <p className="text-gray-400 text-sm mb-4">
        Get notified when we analyze new DAOs. Weekly updates, no spam.
      </p>

      <form onSubmit={handleSubmit} className="flex gap-2">
        <Input
          type="email"
          value={email}
          onChange={(e) => setEmail(e.target.value)}
          placeholder="your@email.com"
          disabled={isSubmitting}
          className="flex-1"
          required
        />
        <Button type="submit" disabled={isSubmitting}>
          {isSubmitting ? 'Subscribing...' : 'Subscribe'}
        </Button>
      </form>

      {result && (
        <div
          className={`mt-3 p-2 rounded text-sm ${
            result.type === 'success'
              ? 'bg-green-900/30 text-green-400'
              : 'bg-red-900/30 text-red-400'
          }`}
        >
          {result.message}
        </div>
      )}

      <p className="text-xs text-gray-500 mt-3">
        By subscribing, you agree to our <a href="/privacy" className="underline">Privacy Policy</a>
      </p>
    </div>
  )
}
```

#### Step 7: Weekly Notification Cron Job
Create cron job to send weekly emails:

```typescript
// src/app/api/jobs/weekly-email/route.ts
import { NextRequest, NextResponse } from 'next/server'
import { getDb } from '@/lib/db'
import { emailSubscribers, daos, gvsSnapshots } from '@/lib/db/schema'
import { eq, and, gt } from 'drizzle-orm'
import { sendRankingsUpdateEmail } from '@/lib/email/resend'

export async function POST(request: NextRequest): Promise<NextResponse> {
  // Verify cron secret
  const authHeader = request.headers.get('authorization')
  if (authHeader !== `Bearer ${process.env.CRON_SECRET}`) {
    return NextResponse.json({ error: 'Unauthorized' }, { status: 401 })
  }

  const db = getDb()

  try {
    // Find DAOs added in the last 7 days
    const sevenDaysAgo = new Date(Date.now() - 7 * 24 * 60 * 60 * 1000)
    const newDAOs = await db
      .select({
        name: daos.name,
        gvsScore: gvsSnapshots.gvsScore,
      })
      .from(daos)
      .innerJoin(gvsSnapshots, eq(daos.id, gvsSnapshots.daoId))
      .where(gt(daos.createdAt, sevenDaysAgo))

    if (newDAOs.length === 0) {
      return NextResponse.json({
        success: true,
        message: 'No new DAOs to notify about',
      })
    }

    // Get confirmed subscribers
    const subscribers = await db
      .select()
      .from(emailSubscribers)
      .where(
        and(
          eq(emailSubscribers.confirmed, true),
          eq(emailSubscribers.unsubscribed, false)
        )
      )

    // Send emails
    let sent = 0
    for (const subscriber of subscribers) {
      try {
        await sendRankingsUpdateEmail(
          subscriber.email,
          subscriber.unsubscribeToken,
          newDAOs.map(dao => ({
            name: dao.name,
            score: parseFloat(dao.gvsScore),
            status: parseFloat(dao.gvsScore) > 7 ? 'Healthy' : 'Needs Attention',
          }))
        )
        sent++
      } catch (error) {
        console.error(`Failed to send to ${subscriber.email}:`, error)
      }
    }

    return NextResponse.json({
      success: true,
      newDAOs: newDAOs.length,
      emailsSent: sent,
      subscribersTotal: subscribers.length,
    })
  } catch (error) {
    console.error('[WeeklyEmail] Error:', error)
    return NextResponse.json({
      success: false,
      error: 'Failed to send emails',
    }, { status: 500 })
  }
}
```

Add to Vercel Cron config (vercel.json):
```json
{
  "crons": [
    {
      "path": "/api/jobs/weekly-email",
      "schedule": "0 9 * * 1"
    }
  ]
}
```

**Acceptance Criteria:**
- [ ] Database migration creates `email_subscribers` table
- [ ] Email capture form on rankings page with clear value prop
- [ ] Double opt-in: confirmation email sent after signup
- [ ] User clicks confirmation link → status = confirmed
- [ ] Weekly cron job checks for new DAOs → sends batch email
- [ ] Every email includes functional unsubscribe link
- [ ] Privacy policy link visible on signup form
- [ ] Tested with real email addresses (Mario's email)

---

## Feature 4: Live Data Ticker (A/B TEST) ⏳ PENDING

**Priority:** P2
**Timeline:** 2-3 hours
**Dependencies:** None (can deploy independently)

**User Value:** Shows data freshness, creates "active tool" perception.

**Implementation:**
- Client-side component fetching pre-generated ticker data
- Display random DAO score change every 5 seconds
- Color-coded: aqua (+), red (-)
- Position: Top-right corner, small and subtle
- Feature flag: Easy to disable if metrics decline

**Acceptance Criteria:**
- [ ] Ticker component built with smooth fade transitions
- [ ] Pre-generated data array (no live API calls)
- [ ] Rotates through recent DAO changes every 5 seconds
- [ ] Color-coded: aqua for positive, red for negative
- [ ] Positioned top-right, doesn't obscure main content
- [ ] Deployed behind feature flag
- [ ] Analytics tracking: bounce rate, time on page, scroll depth

**A/B Test Plan:**
- Deploy ticker for 48 hours
- Monitor metrics:
  - Bounce rate (before vs after)
  - Time on page (before vs after)
  - Scroll depth (do users engage with rankings?)
  - Heatmap clicks (is ticker distracting?)

**Decision criteria:**
- ✅ **Keep ticker if:** Metrics improve or stay neutral + positive user feedback
- ❌ **Remove ticker if:** Bounce rate increases >5% or time-on-page decreases

---

## Environment Variables Needed

Add to `.env.local`:
```
RESEND_API_KEY=re_xxxxx
NEXT_PUBLIC_BASE_URL=https://chainsights.one
CRON_SECRET=random_secret_for_cron_auth
```

---

## Testing Checklist

### Feature 2 (Privacy Policy):
- [ ] Privacy policy page loads without errors
- [ ] Email notifications section visible
- [ ] Links to Resend privacy policy work
- [ ] Deployed BEFORE Feature 3 goes live

### Feature 3 (Email System):
- [ ] Signup form appears on /rankings
- [ ] Submit email → confirmation email received
- [ ] Click confirmation link → redirects to /rankings with success message
- [ ] Try signing up again → "Already subscribed" error
- [ ] Unsubscribe link in email works
- [ ] Weekly cron job sends emails (test manually first)
- [ ] Emails render correctly in Gmail, Outlook, Apple Mail

### Feature 4 (Ticker):
- [ ] Ticker appears in top-right corner
- [ ] Data rotates every 5 seconds
- [ ] Positive changes show in aqua
- [ ] Negative changes show in red
- [ ] Feature flag works (can disable instantly)
- [ ] Analytics tracking fires correctly

---

## Deployment Order

**Tonight (Dec 24):**
1. Fix DATABASE_URL in .env.local (remove line break)
2. Run `npx tsx scripts/add-community-links.ts`
3. Verify community links appear on /rankings
4. Deploy Feature 2 (Privacy Policy) to production

**This Week (Dec 25-27):**
5. Implement Feature 3 (Email System) - 4-6 hours
6. Test with Mario's email address
7. Deploy to production

**Next Week (Dec 30-Jan 3):**
8. Implement Feature 4 (Ticker) - 2-3 hours
9. Deploy behind feature flag
10. Run 48-hour A/B test
11. Decide keep/remove based on metrics

---

## Questions for Mario

1. **Resend Account:** Do you have a Resend account? Need API key.
2. **Email Domain:** Should emails come from `notifications@chainsights.one`?
3. **Cron Auth:** Need CRON_SECRET for securing cron endpoints
4. **Safe DAO Discord:** Couldn't find official Discord link - verify from safe.global?

---

## Success Metrics (from Planning Doc)

**Feature 1 (Discord Links):**
- Success = Click-through rate to Discord/Forum >5%

**Feature 2 (Privacy Policy):**
- Success = GDPR compliant (no violations)

**Feature 3 (Email Capture):**
- Success = 50+ subscribers in first month
- Open rate >30% on first email
- Unsubscribe rate <5%

**Feature 4 (Ticker):**
- Success = Bounce rate stays flat or improves
- Time on page increases or stays neutral

---

**Handoff Complete**
Mary → Amelia
December 24, 2025
