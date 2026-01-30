# Feature Specification: Tiered Reports & Social Sharing

**Document Version:** 1.0
**Date:** 2024-12-20
**Status:** Implementation Planning
**Owner:** Mario

---

## Executive Summary

This specification outlines two interconnected features designed to increase conversion and viral growth:

1. **Tiered Report Offering** - Restructured checkout flow with free tier entry point
2. **Social Sharing Integration** - Twitter share functionality for viral distribution

**Core Problem:** Users don't click "Get Report" because they don't understand what they'll receive.

**Solution:** Offer a free "Quick Health Check" that provides immediate value while creating upgrade path to paid reports.

---

## Feature 1: Tiered Report Offering

### Overview

Restructure the report acquisition flow to provide three distinct tiers with a free entry point that requires email/wallet registration.

### User Flows

#### Flow A: First-Time User - Free Rating Query

1. **Entry Point**
   - User lands on homepage
   - Sees prominent "Get Report" CTA button

2. **DAO Selection**
   - Click "Get Report" → Modal opens
   - User selects DAO from dropdown
   - System performs snapshot availability check in background

3. **Tier Selection**
   - If data available → Display three pricing tiers (see Pricing Tiers section)
   - If no data → Display message: "No governance data available for [DAO Name] yet. [Request Analysis]"

4. **Free Tier Registration**
   - User selects "Quick Health Check" (free option)
   - Form appears with:
     - Email OR wallet address input field
     - Consent checkbox (unchecked by default)
     - "Get Free Score" submit button

5. **Rating Display**
   - Rating appears immediately in modal:
     - Overall Score: X/10
     - Gini Coefficient: 0.XX
     - Nakamoto Coefficient: XXX
     - Current ranking position
   - Simultaneous email delivery with same data

6. **Social Sharing Prompt**
   - Below rating: "Share your DAO's score"
   - Social share buttons displayed (Twitter, LinkedIn)

7. **Upgrade Prompt**
   - "Want to know why? Upgrade to Standard Report ($99)"

#### Flow B: Paid Report Purchase

1-3. Same as Flow A
4. User selects "$99 Standard Report" OR "$199 Deep Dive Analysis"
5. Modal transitions to payment selection (Stripe/Crypto)
6. Complete payment → Full report generated
7. Social sharing prompt with report results

#### Flow C: Returning User - Quota Exhausted

1-3. Same as Flow A
4. User selects "Quick Health Check"
5. System checks quota:
   - `SELECT COUNT(*) FROM free_query_log WHERE identifier_hash = ? AND month_key = ?`
6. If quota used → Display:
   ```
   You've used your free rating for this month
   Resets on [First day of next month]

   Get unlimited access:
   • Standard Report - $99
   • Deep Dive Analysis - $199
   ```

---

### Pricing Tiers

#### Tier 1: Quick Health Check (FREE)

**Positioning:** Entry point for new users, lead generation

**Title:** Quick Health Check
**Subtitle:** Get your governance score instantly

**What's Included:**
- Overall Governance Score (X/10)
- Gini Coefficient
- Nakamoto Coefficient
- Current ranking position among all DAOs

**Delivery:**
- Immediate display in modal
- Email delivery to registered address

**Limitations:**
- 1 free query per month per email/wallet
- Limited to core metrics only

**CTA Button:** "Get Free Score"
**Fine Print:** "1 free per month • Email required"

---

#### Tier 2: Standard Report ($99)

**Positioning:** Most popular option for serious DAO members/researchers

**Title:** Standard Report
**Subtitle:** Understand the 'why' behind your score
**Badge:** "Most Popular"

**What's Included:**
- Everything in Quick Health Check
- Detailed breakdown by governance category
- Historical trends (3-6 months)
- Comparison to similar DAOs
- Key strengths and weaknesses
- Downloadable PDF report

**CTA Button:** "Get Standard Report"

---

#### Tier 3: Deep Dive Analysis ($199)

**Positioning:** Premium option for DAOs seeking improvement

**Title:** Deep Dive Analysis
**Subtitle:** Get actionable recommendations
**Badge:** "Most Comprehensive"

**What's Included:**
- Everything in Standard Report
- Risk assessment and threat analysis
- Improvement roadmap with prioritized recommendations
- Competitive positioning analysis
- Governance best practices comparison
- 30-day update included

**CTA Button:** "Get Deep Dive"

---

### Technical Requirements

#### Frontend Components

**1. Homepage CTA Update**
- Existing "Get Report" button → Update to trigger new modal
- File: `src/app/page.tsx` (or wherever current CTA lives)

**2. Report Modal Component** (NEW)
- Component: `<ReportSelectionModal>`
- Location: `src/components/report-selection-modal.tsx`
- Responsibilities:
  - DAO selection dropdown
  - Snapshot availability check
  - Three-tier pricing display
  - Progressive form disclosure
  - Rating display (for free tier)
  - Social sharing buttons

**3. Pricing Tier Cards** (NEW)
- Component: `<PricingTierCard>`
- Location: `src/components/pricing-tier-card.tsx`
- Props: `tier`, `price`, `features`, `ctaText`, `badge`, `onSelect`

**4. Free Tier Form** (NEW)
- Component: `<FreeRatingForm>`
- Location: `src/components/free-rating-form.tsx`
- Fields:
  - Email or Wallet input (with toggle/auto-detection)
  - Consent checkbox with Privacy Policy link
  - Submit button
- Validation:
  - Email: RFC 5322 compliant
  - Wallet: Ethereum address format (0x...)
  - Consent: Must be checked to proceed

**5. Rating Display** (NEW)
- Component: `<RatingDisplay>`
- Location: `src/components/rating-display.tsx`
- Shows: Score, Gini, Nakamoto, ranking
- Includes upgrade CTA

#### Backend API Endpoints

**1. Snapshot Availability Check**
```
GET /api/dao/snapshot-available?dao={daoName}

Response:
{
  "available": boolean,
  "lastSnapshot": "2024-12-15T10:00:00Z" | null,
  "dataQuality": "good" | "partial" | "insufficient"
}
```

**2. Free Tier Registration & Rating**
```
POST /api/reports/free-rating

Request Body:
{
  "dao": "string",
  "identifier": "string", // email or wallet
  "identifierType": "email" | "wallet",
  "consentGiven": true,
  "consentTimestamp": "2024-12-20T14:30:00Z"
}

Response:
{
  "success": boolean,
  "rating": {
    "score": 7.2,
    "giniCoefficient": 0.45,
    "nakamotoCoefficient": 850,
    "ranking": 42,
    "totalDaos": 150
  },
  "quotaRemaining": 0,
  "nextResetDate": "2025-01-01T00:00:00Z"
}

Error Response (Quota Exhausted):
{
  "success": false,
  "error": "QUOTA_EXCEEDED",
  "nextResetDate": "2025-01-01T00:00:00Z"
}
```

**3. Quota Check**
```
GET /api/reports/quota-check?identifier={hash}&month={YYYY-MM}

Response:
{
  "quotaUsed": 1,
  "quotaLimit": 1,
  "nextResetDate": "2025-01-01T00:00:00Z"
}
```

#### Database Schema

**New Table: `free_query_log`**

```sql
CREATE TABLE free_query_log (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  identifier_hash VARCHAR(64) NOT NULL, -- SHA-256 hash of email/wallet
  identifier_type VARCHAR(10) NOT NULL CHECK (identifier_type IN ('email', 'wallet')),
  dao_id VARCHAR(255) NOT NULL,
  rating_score DECIMAL(3,1) NOT NULL,
  gini_coefficient DECIMAL(4,3),
  nakamoto_coefficient INT,
  ranking_position INT,
  consent_given BOOLEAN NOT NULL DEFAULT false,
  consent_timestamp TIMESTAMP NOT NULL,
  created_at TIMESTAMP NOT NULL DEFAULT NOW(),
  month_key VARCHAR(7) NOT NULL, -- Format: 'YYYY-MM' for easy quota queries
  email_sent BOOLEAN DEFAULT false,
  email_sent_at TIMESTAMP,

  -- Indexes for performance
  INDEX idx_identifier_month (identifier_hash, month_key),
  INDEX idx_dao_created (dao_id, created_at),
  INDEX idx_month_key (month_key),

  -- Ensure one free query per month per identifier
  UNIQUE KEY unique_monthly_query (identifier_hash, month_key)
);
```

**Existing Table Updates:**

Check if `daos` or equivalent table needs:
- `included_in_rankings` BOOLEAN field (to track opt-in from free tier)

#### Backend Logic Requirements

**1. Identifier Hashing**
- Hash all emails/wallets before storage using SHA-256
- Never store plain-text email/wallet addresses
- Use consistent hashing function across application

**2. Quota Enforcement**
```typescript
// Pseudocode
async function checkQuota(identifier: string, identifierType: string): Promise<QuotaStatus> {
  const hash = sha256(identifier.toLowerCase().trim());
  const monthKey = getCurrentMonthKey(); // "2024-12"

  const existingQuery = await db.query(
    'SELECT * FROM free_query_log WHERE identifier_hash = ? AND month_key = ?',
    [hash, monthKey]
  );

  return {
    quotaUsed: existingQuery.length,
    quotaLimit: 1,
    canQuery: existingQuery.length === 0,
    nextReset: getFirstDayOfNextMonth()
  };
}
```

**3. Rating Generation**
- Reuse existing rating calculation logic from paid reports
- Extract: overall score, Gini, Nakamoto, ranking
- Ensure data is current (use latest snapshot)

**4. Email Delivery**
- Use existing email service (transactional email provider)
- Template: See Email Templates section
- Include unsubscribe mechanism (best practice)

**5. Rankings Integration**
- Free tier ratings should feed into public rankings
- Consent flag determines inclusion: `consent_given = true`
- Update rankings calculation job to include free tier data

---

### Email Templates

#### Free Rating Delivery Email

**Subject:** `[DAO_NAME] Governance Rating: [SCORE]/10`

**Body (HTML):**
```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8">
  <title>Your DAO Governance Rating</title>
</head>
<body style="font-family: Arial, sans-serif; max-width: 600px; margin: 0 auto;">

  <h1 style="color: #333;">Your DAO Governance Rating</h1>

  <div style="background: #f5f5f5; padding: 20px; border-radius: 8px; margin: 20px 0;">
    <h2 style="margin-top: 0;">[DAO_NAME]</h2>
    <p style="font-size: 32px; font-weight: bold; margin: 10px 0;">[SCORE]/10</p>

    <table style="width: 100%; margin-top: 20px;">
      <tr>
        <td><strong>Gini Coefficient:</strong></td>
        <td>[GINI]</td>
      </tr>
      <tr>
        <td><strong>Nakamoto Coefficient:</strong></td>
        <td>[NAKAMOTO]</td>
      </tr>
      <tr>
        <td><strong>Ranking:</strong></td>
        <td>#[RANK] of [TOTAL] DAOs</td>
      </tr>
    </table>
  </div>

  <p>Want to understand <strong>why</strong> this is your score?</p>

  <a href="[UPGRADE_URL]" style="display: inline-block; background: #0066cc; color: white; padding: 12px 24px; text-decoration: none; border-radius: 4px; margin: 10px 0;">
    Get Standard Report - $99
  </a>

  <hr style="margin: 30px 0; border: none; border-top: 1px solid #ddd;">

  <p style="font-size: 12px; color: #666;">
    [COMPANY_NAME]<br>
    [ADDRESS]<br>
    <a href="[PRIVACY_POLICY_URL]">Privacy Policy</a> |
    <a href="[UNSUBSCRIBE_URL]">Unsubscribe</a>
  </p>

</body>
</html>
```

**Plain Text Version:**
```
Your DAO Governance Rating
==========================

[DAO_NAME]: [SCORE]/10

Gini Coefficient: [GINI]
Nakamoto Coefficient: [NAKAMOTO]
Ranking: #[RANK] of [TOTAL] DAOs

Want to understand WHY this is your score?

Get Standard Report ($99): [UPGRADE_URL]

---
[COMPANY_NAME]
[ADDRESS]
Privacy Policy: [PRIVACY_POLICY_URL]
Unsubscribe: [UNSUBSCRIBE_URL]
```

---

## Feature 2: Social Sharing Integration

### Overview

Enable users to share their DAO ratings on Twitter (and optionally LinkedIn) with pre-filled, shareable content that drives viral growth.

### User Experience

**Trigger Points:**
1. Immediately after receiving free rating (in modal)
2. After purchasing/viewing paid report (on report page)
3. Persistent "Share" button on any rating/report view

**Share Flow:**
1. User clicks "Share on Twitter" button
2. Pop-up window opens (600x400px) with Twitter Web Intent
3. Pre-filled tweet appears (user can edit)
4. User posts or cancels
5. If posted → Optional analytics tracking (if user consented)

### Technical Implementation

#### Frontend Components

**1. Social Share Button Component** (NEW)
- Component: `<SocialShareButton>`
- Location: `src/components/social-share-button.tsx`
- Props:
  ```typescript
  interface SocialShareButtonProps {
    platform: 'twitter' | 'linkedin' | 'farcaster';
    daoName: string;
    score: number;
    gini?: number;
    nakamoto?: number;
    reportUrl: string;
    userType?: 'dao_member' | 'researcher' | 'general';
  }
  ```

**2. Tweet Generator Function**
- Location: `src/lib/social-share.ts`
- Function: `generateTweet(params: TweetParams): string`
- Handles dynamic text generation based on user type

#### Twitter Web Intent Integration

**No Twitter App Required** - Use Twitter Web Intents (simple URL scheme):

```typescript
function openTwitterShare(text: string, url: string) {
  const tweetUrl = `https://twitter.com/intent/tweet?text=${encodeURIComponent(text)}&url=${encodeURIComponent(url)}`;

  window.open(
    tweetUrl,
    'twitter-share',
    'width=600,height=400,toolbar=no,menubar=no,scrollbars=yes'
  );
}
```

**Pre-filled Tweet Templates:**

**For DAO Members:**
```
Just checked our governance health on @[YOUR_TWITTER_HANDLE] 📊

[DAO_NAME]: [SCORE]/10
Decentralization: Gini [GINI]

See how your DAO ranks: [URL]
```

**For Researchers/General Users:**
```
Analyzed [DAO_NAME] governance using @[YOUR_TWITTER_HANDLE]:

Score: [SCORE]/10
Nakamoto Coefficient: [NAKAMOTO]

Check any DAO instantly: [URL]
```

**Character Limits:**
- Max tweet length: 280 characters
- Reserve ~25 characters for URL (t.co shortening)
- Ensure templates stay under 255 characters pre-URL

#### Optional: LinkedIn Sharing

**LinkedIn Share URL:**
```
https://www.linkedin.com/sharing/share-offsite/?url=[ENCODED_URL]
```

Note: LinkedIn doesn't support pre-filled text via URL params. Consider:
- Copy-to-clipboard functionality with suggested text
- OR simpler: Just share the URL and let LinkedIn scrape Open Graph tags

#### Optional: Farcaster Integration

For Web3-native audience, consider Farcaster sharing via Warpcast:
```
https://warpcast.com/~/compose?text=[ENCODED_TEXT]&embeds[]=[URL]
```

### Open Graph Meta Tags

**Critical:** Ensure report pages have proper Open Graph tags for rich social previews:

**Required meta tags in report pages:**
```html
<meta property="og:title" content="[DAO_NAME] Governance Report - Score: [SCORE]/10">
<meta property="og:description" content="Comprehensive governance analysis: Gini [GINI], Nakamoto [NAKAMOTO]. See detailed breakdown.">
<meta property="og:image" content="[URL_TO_SHARE_IMAGE]">
<meta property="og:url" content="[CANONICAL_REPORT_URL]">
<meta property="og:type" content="article">
<meta name="twitter:card" content="summary_large_image">
<meta name="twitter:site" content="@[YOUR_HANDLE]">
```

**Share Image Generation:**
Consider dynamic Open Graph image generation:
- Service: Vercel OG Image or similar
- Template: DAO name + score + key metrics
- Endpoint: `/api/og-image?dao=[NAME]&score=[SCORE]`

### Analytics Tracking (Optional)

**Track share events:**
```typescript
// When share button clicked
analytics.track('social_share_initiated', {
  platform: 'twitter',
  dao_name: daoName,
  score: score,
  user_type: userType
});

// Track via UTM parameters in shared URL
const shareUrl = `${baseUrl}?utm_source=twitter&utm_medium=social&utm_campaign=user_share&utm_content=${daoName}`;
```

---

## Legal & Compliance Requirements

### GDPR Compliance Checklist

#### Data Collection (Free Tier)

✅ **Consent Mechanism**
- [ ] Consent checkbox implemented (unchecked by default)
- [ ] Checkbox label: "I agree to receive my free monthly DAO rating and contribute to public rankings"
- [ ] Privacy Policy link adjacent to checkbox
- [ ] Cannot submit without explicit consent
- [ ] Consent timestamp captured and stored

✅ **Privacy Policy Updates**
- [ ] Add section: "Free DAO Rating Service"
- [ ] Explain data collected: email/wallet, monthly quota tracking, rating data
- [ ] State purpose: Delivering ratings, enforcing quota, contributing to rankings
- [ ] Data retention: "Until deletion request or 12 months of inactivity"
- [ ] Third parties: Mention email service provider (if applicable)
- [ ] User rights: Access, deletion, portability

✅ **Data Minimization**
- [ ] Store only hashed identifiers (SHA-256)
- [ ] No unnecessary personal data collection
- [ ] No tracking without consent

✅ **User Rights Implementation**
- [ ] Right to Access: Provide endpoint/email for users to request their data
- [ ] Right to Deletion: Process deletion requests within 30 days
- [ ] Right to Portability: Export user's rating history in JSON/CSV format

#### Email Communications

✅ **Transactional Email Requirements**
- [ ] Subject line clearly states purpose
- [ ] Sender name and address included in footer
- [ ] Privacy Policy link in footer
- [ ] Unsubscribe mechanism (best practice, even for transactional)
- [ ] No marketing content without separate opt-in

#### Social Sharing

✅ **User Control**
- [ ] No auto-posting - explicit user click required
- [ ] Pre-filled text must be editable (Twitter enforces this)
- [ ] Clear indication that clicking will open external platform

✅ **Data Sharing**
- [ ] No sharing of user data with Twitter beyond public post content
- [ ] No tracking pixels without consent
- [ ] Privacy Policy mentions "social sharing features"

### Database Compliance

**Stored Data Audit:**

| Field | Personal Data? | Basis | Retention |
|-------|---------------|-------|-----------|
| `identifier_hash` | Pseudonymous | Consent | 12 months or deletion request |
| `identifier_type` | No | N/A | Same as above |
| `dao_id` | No | N/A | Permanent (anonymized) |
| `rating_score` | No | N/A | Permanent (anonymized) |
| `gini_coefficient` | No | N/A | Permanent (anonymized) |
| `nakamoto_coefficient` | No | N/A | Permanent (anonymized) |
| `consent_given` | No | Legal requirement | Same as identifier |
| `consent_timestamp` | No | Legal requirement | Same as identifier |

**Data Deletion Process:**
1. User requests deletion via email/web form
2. Identify all records via identifier hash
3. Delete from `free_query_log` table
4. Remove from email distribution lists
5. Confirm deletion to user within 30 days
6. Keep anonymized aggregate data (scores, rankings) - GDPR compliant

---

## Implementation Phases

### Phase 1: Foundation (Priority: HIGH)

**Goal:** Enable DAO selection and snapshot checking

**Tasks:**
1. Update homepage "Get Report" CTA to trigger new modal
2. Create `ReportSelectionModal` component
3. Implement DAO dropdown (reuse existing DAO list)
4. Build snapshot availability check API endpoint
5. Create loading states and error handling

**Deliverables:**
- [ ] Updated homepage with new CTA behavior
- [ ] Modal opens and displays DAO selection
- [ ] Snapshot check displays availability status

**Estimated Effort:** 2-3 days

---

### Phase 2: Tiered Pricing Display (Priority: HIGH)

**Goal:** Present three pricing tiers after DAO selection

**Tasks:**
1. Create `PricingTierCard` component
2. Implement tier comparison layout
3. Add copy/content for each tier (see Pricing Tiers section)
4. Handle tier selection logic
5. Add "No data available" state

**Deliverables:**
- [ ] Three pricing tiers displayed after DAO selection
- [ ] Cards show pricing, features, CTAs
- [ ] Selection triggers appropriate flow (free form or payment)

**Estimated Effort:** 2 days

---

### Phase 3: Free Tier - Registration & Display (Priority: HIGH)

**Goal:** Allow users to register and receive free ratings

**Tasks:**
1. Create `FreeRatingForm` component
   - Email/wallet input field
   - Consent checkbox with Privacy Policy link
   - Validation logic
2. Build `free_query_log` database table
3. Implement identifier hashing utility
4. Create `/api/reports/free-rating` endpoint
   - Quota checking
   - Rating calculation
   - Database logging
5. Create `RatingDisplay` component
6. Implement email delivery service
7. Design and code email template

**Deliverables:**
- [ ] Free tier form accepts email/wallet + consent
- [ ] Database stores hashed identifier + consent timestamp
- [ ] Rating displayed immediately after submission
- [ ] Email sent with rating details
- [ ] Quota enforcement working (1 per month)

**Estimated Effort:** 4-5 days

---

### Phase 4: Social Sharing Integration (Priority: MEDIUM)

**Goal:** Enable Twitter sharing of ratings

**Tasks:**
1. Create `SocialShareButton` component
2. Implement tweet generation logic with templates
3. Add Twitter Web Intent integration
4. Place share buttons in:
   - Free rating display modal
   - Paid report pages (if not already present)
5. Set up Open Graph meta tags for report pages
6. Optional: Add LinkedIn/Farcaster share buttons
7. Optional: Implement share analytics tracking

**Deliverables:**
- [ ] Twitter share button appears after rating display
- [ ] Clicking opens Twitter with pre-filled text
- [ ] Text includes DAO name, score, and URL
- [ ] Open Graph tags provide rich previews

**Estimated Effort:** 3-4 days

---

### Phase 5: Quota Management & Error States (Priority: MEDIUM)

**Goal:** Handle edge cases and quota exhaustion gracefully

**Tasks:**
1. Implement quota check on free tier selection
2. Create "quota exceeded" UI state with upgrade prompt
3. Add "next reset date" display
4. Handle concurrent requests (database constraint)
5. Create admin endpoint to view/reset quotas (optional)
6. Add error logging and monitoring

**Deliverables:**
- [ ] Returning users see quota status
- [ ] Clear messaging when quota exhausted
- [ ] Upgrade prompts display correctly
- [ ] Database constraints prevent quota bypass

**Estimated Effort:** 2-3 days

---

### Phase 6: Rankings Integration (Priority: LOW)

**Goal:** Include free tier ratings in public rankings

**Tasks:**
1. Update ranking calculation job to query `free_query_log`
2. Filter for `consent_given = true` records
3. Aggregate free + paid rating data
4. Update public rankings display
5. Add "based on X reports" transparency indicator

**Deliverables:**
- [ ] Free tier ratings contribute to rankings
- [ ] Public rankings reflect combined data
- [ ] Transparency about data sources

**Estimated Effort:** 2 days

---

### Phase 7: Privacy Policy & Legal Updates (Priority: HIGH)

**Goal:** Ensure legal compliance before launch

**Tasks:**
1. Draft Privacy Policy updates (consult legal counsel)
2. Add "Free DAO Rating Service" section
3. Update data retention policies
4. Implement deletion request process
5. Add unsubscribe mechanism to emails
6. Document compliance procedures

**Deliverables:**
- [ ] Updated Privacy Policy published
- [ ] Deletion request process documented
- [ ] Compliance checklist completed

**Estimated Effort:** 1-2 days (assuming legal review is expedited)

---

### Phase 8: Testing & QA (Priority: HIGH)

**Goal:** Ensure all flows work correctly

**Test Cases:**
1. **Free Tier - First Time User**
   - Select DAO → Choose free tier → Enter email → Receive rating
   - Verify email delivery
   - Verify database record created
   - Test consent checkbox validation

2. **Free Tier - Quota Exhausted**
   - Attempt second free query in same month
   - Verify quota exceeded message
   - Verify upgrade prompt displays

3. **Free Tier - Next Month Reset**
   - Query after month rollover
   - Verify quota resets correctly

4. **Paid Tier Flow**
   - Select paid tier → Verify payment flow unchanged
   - Ensure no interference with existing checkout

5. **Social Sharing**
   - Click Twitter share → Verify popup opens
   - Check pre-filled text format
   - Test URL encoding
   - Verify Open Graph tags render correctly

6. **Error States**
   - No snapshot available
   - Invalid email format
   - Network errors
   - Concurrent quota attempts

7. **Cross-Browser Testing**
   - Chrome, Firefox, Safari, Edge
   - Mobile browsers (iOS Safari, Chrome Mobile)

8. **Privacy Compliance**
   - Consent checkbox cannot be bypassed
   - Privacy Policy link works
   - Unsubscribe link in email works
   - Hashing function produces consistent results

**Deliverables:**
- [ ] All test cases pass
- [ ] Bug fixes completed
- [ ] Performance validated (modal load time < 1s)

**Estimated Effort:** 3-4 days

---

### Phase 9: Launch & Monitoring (Priority: HIGH)

**Goal:** Deploy features and monitor performance

**Tasks:**
1. Deploy to staging environment
2. Conduct final smoke tests
3. Deploy to production
4. Monitor error rates and performance
5. Track conversion metrics:
   - Free tier sign-ups
   - Free → Paid conversion rate
   - Social shares per rating
6. Gather user feedback

**Deliverables:**
- [ ] Features live in production
- [ ] Analytics dashboards configured
- [ ] Error monitoring active
- [ ] Conversion tracking working

**Estimated Effort:** 1-2 days deployment + ongoing monitoring

---

## Success Metrics

### Primary KPIs

1. **Free Tier Adoption**
   - Target: 100+ free ratings in first month
   - Measure: `SELECT COUNT(*) FROM free_query_log WHERE created_at >= '2025-01-01'`

2. **Free → Paid Conversion**
   - Target: 5-10% conversion rate
   - Measure: Track users who get free rating then purchase within 30 days

3. **Social Sharing Rate**
   - Target: 20% of free tier users share on Twitter
   - Measure: Share button click rate + UTM parameter tracking

4. **Viral Coefficient**
   - Target: Each share brings 0.5+ new visitors
   - Measure: UTM-tagged traffic from social shares

### Secondary Metrics

- Email open rate for free tier delivery emails (target: 40%+)
- Time from "Get Report" click to rating display (target: < 15 seconds)
- Quota exhaustion rate (% of users who return for 2nd query)
- Rankings data growth (% increase in DAO coverage)

---

## Open Questions & Decisions Needed

### High Priority

1. **Email Service Provider**
   - Question: Which service for transactional emails? (Sendgrid, Postmark, AWS SES?)
   - Decision Maker: Mario + Dev Team
   - Impact: Integration effort varies

2. **DAO Snapshot Data**
   - Question: What's minimum data quality threshold for showing free tier?
   - Decision Maker: Mario + Data Team
   - Impact: Affects "no data available" frequency

3. **Twitter Handle**
   - Question: What's the official Twitter handle to use in pre-filled tweets?
   - Decision Maker: Mario
   - Impact: Branding consistency

### Medium Priority

4. **Dynamic OG Images**
   - Question: Should we generate dynamic share images or use static template?
   - Decision Maker: Mario + Pixel (designer)
   - Impact: Viral appeal vs implementation effort

5. **LinkedIn/Farcaster**
   - Question: Implement in Phase 4 or defer to Phase 10?
   - Decision Maker: Mario + Maya (marketing)
   - Impact: Scope of initial launch

6. **Admin Dashboard**
   - Question: Need admin UI to view free tier usage stats?
   - Decision Maker: Mario
   - Impact: +2 days development if yes

### Low Priority

7. **A/B Testing**
   - Question: Should we A/B test tier pricing/messaging?
   - Decision Maker: Maya (marketing)
   - Impact: Requires analytics instrumentation

8. **Referral Tracking**
   - Question: Track which social shares convert to sign-ups?
   - Decision Maker: Mario + Marketing
   - Impact: More complex UTM parameter handling

---

## Risk Assessment

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|------------|
| Users abuse free tier with multiple emails | Medium | Low | Rate limiting by IP, email verification |
| Quota logic has race conditions | Low | Medium | Use database UNIQUE constraint + transaction isolation |
| Email deliverability issues | Medium | High | Use reputable ESP, warm up sending domain, monitor bounce rates |
| Twitter changes Web Intent API | Low | Medium | Monitor Twitter developer updates, have fallback to manual copy-paste |
| GDPR compliance gaps | Low | High | Legal counsel review before launch, document all data practices |
| Free tier cannibalizes paid sales | Medium | Medium | Monitor conversion rates, adjust free tier features if needed |
| No snapshot data for popular DAOs | High | High | Prioritize data collection for top 50 DAOs before launch |
| Users don't understand "Quick Health Check" value | Medium | High | A/B test messaging, add example rating in modal before selection |

---

## Appendix

### Wireframe Descriptions

*(Note: Pixel designer to create actual wireframes)*

**Modal Flow:**

```
┌─────────────────────────────────────────┐
│  Get Your DAO Governance Report         │
│                                         │
│  Select DAO:  [Dropdown ▼]             │
│                                         │
│  [Checking data availability...]        │
└─────────────────────────────────────────┘

        ↓ After selection ↓

┌─────────────────────────────────────────┐
│  Choose Your Report                      │
│                                         │
│  ┌───────┐ ┌───────┐ ┌────────┐       │
│  │ FREE  │ │  $99  │ │  $199  │       │
│  │Quick  │ │Standard│ │ Deep   │       │
│  │Health │ │Report │ │ Dive   │       │
│  │Check  │ │       │ │Analysis│       │
│  │       │ │ POPULAR│ │        │       │
│  │• Score│ │• Score │ │• Score │       │
│  │• Gini │ │• Detail│ │• Detail│       │
│  │• Rank │ │• Trends│ │• Roadmap│      │
│  │       │ │• Compare│ │• Risks │      │
│  │[Select]│ │[Select]│ │[Select]│      │
│  └───────┘ └───────┘ └────────┘       │
│                                         │
│  1 free/month                          │
└─────────────────────────────────────────┘

        ↓ Free tier selected ↓

┌─────────────────────────────────────────┐
│  Quick Health Check - Free              │
│                                         │
│  Email or Wallet Address:               │
│  [___________________________]         │
│                                         │
│  ☐ I agree to receive my free monthly  │
│     rating and contribute to rankings  │
│     Privacy Policy                     │
│                                         │
│  [Get Free Score]                      │
└─────────────────────────────────────────┘

        ↓ After submission ↓

┌─────────────────────────────────────────┐
│  Uniswap Governance Rating              │
│                                         │
│         7.2 / 10                       │
│                                         │
│  Gini Coefficient: 0.45                │
│  Nakamoto Coefficient: 850              │
│  Ranking: #42 of 150 DAOs              │
│                                         │
│  ─────────────────────────────         │
│                                         │
│  Want to know WHY?                     │
│  [Get Standard Report - $99]           │
│                                         │
│  Share your score:                     │
│  [🐦 Twitter] [💼 LinkedIn]           │
│                                         │
│  Rating sent to your email ✓          │
└─────────────────────────────────────────┘
```

### Related Documents

- Privacy Policy (to be updated)
- Existing report generation logic (reference)
- Current Stripe/Crypto checkout flow (maintain compatibility)
- Email service provider documentation

### Revision History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2024-12-20 | Paige (Tech Writer) | Initial specification created from party mode discussion |

---

**Document Status: READY FOR REVIEW**

**Next Steps:**
1. Mario reviews and approves specification
2. Address open questions (especially email provider, Twitter handle)
3. Dev team estimates implementation timeline
4. Legal counsel reviews Privacy Policy updates
5. Begin Phase 1 implementation

---

*This document created by Paige (Technical Writer) with input from Sally (UX), Winston (Architect), Maya (Marketing), and Daniel (Legal) in Party Mode collaboration.*
