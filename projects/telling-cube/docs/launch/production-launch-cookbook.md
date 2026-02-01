# tellingCube Production Launch Cookbook

**Status:** LAUNCH DAY - 2025-12-17
**Created:** 2025-12-12
**Updated:** 2025-12-17
**Authors:** Sophie (Marketing), Maya (Marketing), Mary (Business Analyst), Winston (Architect), James (Developer)

---

## Best Launch Day & Time (Sophie)

### Recommended: Tuesday or Wednesday, 8:00-10:00 AM CET

**Why Tuesday/Wednesday?**
- LinkedIn engagement peaks Tuesday-Thursday
- Avoids Monday inbox overload
- Avoids Friday "weekend mode"
- B2B audience most active mid-week

**Why 8:00-10:00 AM CET?**
- Catches European morning (your primary timezone)
- Catches US East Coast early morning (5:00-7:00 AM EST)
- Posts gain momentum before noon
- Algorithm favors early engagement

**Alternative:** Thursday 8:00 AM CET (if you need more prep time)

**Avoid:**
- Mondays (inbox overload)
- Fridays (weekend mode)
- Weekends (B2B audience offline)
- Late afternoons (engagement drops)

---

## Launch Day Checklist

### T-24 Hours (Day Before Launch)

#### Connect Domain to Vercel (James)

**Do this FIRST** - DNS propagation can take hours.

1. **Add Domain in Vercel**
   - Go to Vercel Dashboard → Project → Settings → Domains
   - Add `tellingcube.com`
   - Vercel shows required DNS records

2. **Update DNS at Registrar**
   - Option A: A record → `76.76.21.21` (Vercel IP)
   - Option B: CNAME → `cname.vercel-dns.com`
   - Add `www` subdomain redirect (optional)

3. **Verify Connection**
   - [✅] `https://tellingcube.com` loads your app
   - [✅] SSL certificate is valid (padlock icon)
   - [✅] `www.tellingcube.com` redirects to `tellingcube.com`

**Note:** SSL certificate auto-provisions once DNS propagates (usually 10-30 minutes).

#### Launch Proxy Page (James)

**OPTIONAL: Use this to hide the main site before official launch.**

The launch proxy shows a "Coming Soon" page on the production domain while keeping the full site accessible on Vercel preview URLs for testing.

1. **Enable Launch Proxy (Before DNS propagation)**
   - In Vercel Dashboard → Project → Settings → Environment Variables
   - Add: `LAUNCH_PROXY_ENABLED` = `true`
   - Redeploy or wait for auto-redeploy

2. **What Happens**
   - `tellingcube.com` and `www.tellingcube.com` → Shows launch proxy page
   - `*.vercel.app` URLs → Full site (for testing)
   - `localhost:3000` → Full site (for development)

3. **Disable Launch Proxy (Launch Day!)**
   - In Vercel Dashboard → Environment Variables
   - Set `LAUNCH_PROXY_ENABLED` = `false` (or delete the variable)
   - Site goes live immediately (no redeploy needed for env var changes)

4. **Verify**
   - [✅] Proxy shows on `https://tellingcube.com`
   - [✅] Full site accessible on Vercel preview URL
   - [✅] Launch date displays correctly (Wednesday, December 17th 2025)
   - [✅] Contact email link works (`mailto:contact@masem.at`)

**Files:**
- `middleware.ts` - Routes requests to proxy
- `app/launch-proxy/page.tsx` - The proxy page component

#### Search Engine Indexing (James)

**Do after domain is connected** - indexing takes days/weeks to fully propagate.

1. **Google Search Console**
   - Go to https://search.google.com/search-console
   - Add property → URL prefix → `https://tellingcube.com`
   - Verify via DNS TXT record or HTML file upload
   - Submit sitemap: `https://tellingcube.com/sitemap.xml`

2. **Bing Webmaster Tools**
   - Go to https://www.bing.com/webmasters
   - Import from Google Search Console (easiest) or add manually
   - Verify ownership
   - Sitemap auto-imports from Google or submit manually

3. **Verify Setup**
   - [✅] Google Search Console shows "Sitemap submitted"
   - [✅] Bing Webmaster Tools shows site verified
   - [✅] No crawl errors reported

#### Stripe Production Mode Switch (Winston)

**Do this FIRST on T-24h** - gives time to verify before launch.

1. **Get Production Keys from Stripe**
   - Go to https://dashboard.stripe.com/apikeys
   - Toggle OFF "Test mode" (top right)
   - Copy `sk_live_...` secret key

2. **Create Production Webhook**
   - Go to https://dashboard.stripe.com/webhooks
   - Click "Add endpoint"
   - URL: `https://tellingcube.com/api/stripe/webhook`
   - Events: Select `checkout.session.completed`
   - Click "Add endpoint"
   - Copy the signing secret (`whsec_...`)

3. **Update Vercel Environment Variables**
   ```
   STRIPE_SECRET_KEY=sk_live_...      (replace sk_test_...)
   STRIPE_WEBHOOK_SECRET=whsec_...    (new production webhook secret)
   ```
   Note: Vercel auto-redeploys when env vars change

4. **Verify Production Payment**
   - [✅] Make a real purchase (use your own card)
   - [✅] Verify payment appears in Stripe dashboard (live mode)
   - [✅] Verify confirmation email arrives
   - [✅] Refund the test purchase in Stripe dashboard

#### Technical (Winston/James)

- [✅] **Vercel Production Check**
  ```bash
  # Verify deployment is healthy
  curl -I https://tellingcube.com
  # Should return 200 OK
  ```

- [✅] **Environment Variables Verified**
  - [✅] `DATABASE_URL` - NeonDB production
  - [✅] `ANTHROPIC_API_KEY` - Valid and has credits
  - [✅] `STRIPE_SECRET_KEY` - Production key (not test!)
  - [✅] `STRIPE_WEBHOOK_SECRET` - Production webhook
  - [✅] `RESEND_API_KEY` - Email service active
  - [✅] `UPSTASH_REDIS_*` - Rate limiting enabled (optional but recommended)
  - [✅] `TURNSTILE_*` - Bot protection enabled

- [✅] **Test Generation Flow**
  1. Generate one scenario end-to-end
  2. Verify events CSV download works
  3. Verify JSON API endpoints work
  4. Verify consistency badge shows green

- [✅] **Test Payment Flow**
  1. Use Stripe test card (if test mode)
  2. Verify email confirmation arrives
  3. Check Stripe dashboard for test transaction

- [✅] **Rate Limiting (if enabled)**
  ```
  GENERATIONS_PER_HOUR=10
  GLOBAL_DAILY_GENERATION_LIMIT=50
  NEXT_PUBLIC_FREE_GENERATIONS_PER_DAY=3
  ```

- [ ] **Monitoring Setup**
  - [ ] Vercel Analytics enabled
  - [ ] Error tracking (Vercel logs)
  - [ ] Stripe webhook monitoring

#### Content Preparation (Sophie/Maya)

- [ ] **LinkedIn Post Ready**
  - Draft finalized (see `docs/marketing/launch/linkedin-posts.md`)
  - No prohibited content (check `docs/marketing/CRITICAL-GUIDELINES.md`)
  - Image/video attached (if using)

- [✅] **Twitter/X Posts Ready**
  - Thread drafted (see `docs/marketing/launch/twitter-posts.md`)
  - Choose variant (A recommended for launch)
  - Video exported for native upload (MP4, 1080p, <512MB)

- [✅] **Demo Video Ready**
  - Recorded and edited (see `docs/marketing/launch/demo-video-script.md`)
  - Thumbnail created (1280x720)
  - **Subtitles added:**
    - [✅] Auto-transcribe with CapCut/Descript
    - [✅] Review and fix errors (check "tellingCube" spelling)
    - [✅] Export hardcoded version for LinkedIn/Twitter
    - [✅] Export SRT file for YouTube (optional)
    - [✅] Test readability on mobile
  - YouTube metadata prepared (see `docs/marketing/launch/youtube-metadata.md`)
    - [✅] Title finalized
    - [✅] Description with timestamps copied
    - [✅] Tags added
    - [✅] Thumbnail uploaded
    - [✅] Category: Science & Technology
    - [✅] End screen configured
  - Video exported for Twitter native upload (separate from YouTube)
  - Video uploaded natively to LinkedIn (for better reach)

- [✅] **Responses Prepared**
  - FAQ answers ready
  - Pricing objection responses
  - Technical question answers
  - Twitter reply templates ready (see twitter-posts.md)

#### Domain & DNS (James)

- [✅] **Domain Active**
  - `tellingcube.com` resolves correctly
  - SSL certificate valid (https works)
  - No DNS propagation issues

- [✅] **Redirects Working**
  - `www.tellingcube.com` → `tellingcube.com`
  - Old URLs redirect properly (if any)

---

### T-1 Hour (Launch Morning)

#### Final Checks (All)

- [✅] **Site Accessible**
  ```bash
  curl -I https://tellingcube.com
  # Verify 200 OK
  ```

- [✅] **Quick Generation Test**
  - Generate a Startup scenario
  - Takes ~2 minutes
  - Downloads work

- [✅] **Payment Test** (if not done yesterday)
  - Test card works
  - Email arrives

- [✅] **Social Accounts Ready**
  - LinkedIn logged in
  - Post copied and ready to paste

---

### T-0 (Launch!)

#### Step 1: Go Live Announcement (Sophie/Maya)

1. **Post to LinkedIn** (8:00-10:00 AM CET)
   - Use Variant A (Founder Story) for launch
   - Post from personal profile (not company page)
   - DO NOT schedule - post manually for best engagement

2. **Post to Twitter/X** (same time or +30 min)
   - Use Variant A thread (7 tweets) from `docs/marketing/launch/twitter-posts.md`
   - Post thread manually (not scheduled)
   - Reply to own last tweet with native video upload
   - Self-reply with engagement hook (see twitter-posts.md)

3. **Upload to YouTube**
   - Use metadata from `docs/marketing/launch/youtube-metadata.md`
   - Set to Public immediately (not Unlisted)
   - Add to "tellingCube Tutorials" playlist

4. **Engage Immediately** (All Platforms)
   - Like your own posts (signals activity)
   - Reply to first 5 comments within 30 minutes
   - Personal responses, not canned replies

#### Step 2: Monitor (James/Winston)

1. **Watch Vercel Logs**
   ```
   vercel logs --follow
   ```

2. **Watch for Errors**
   - Generation failures
   - Payment issues
   - Rate limit hits

3. **Monitor Costs**
   - Claude API usage (Anthropic dashboard)
   - NeonDB connections
   - Vercel function invocations

#### Step 3: Respond to Feedback (All)

- [ ] Reply to all LinkedIn comments within 2 hours
- [ ] Fix any reported bugs immediately
- [ ] Note feature requests for later

---

## Post-Launch (First 48 Hours)

### Hour 1-4
- [ ] Monitor all comments and reply (LinkedIn + Twitter + YouTube)
- [ ] Watch for technical issues
- [ ] Track LinkedIn post metrics
- [ ] Track Twitter thread metrics (impressions, retweets)
- [ ] Check YouTube views and watch time

### Hour 4-24
- [ ] Post follow-up comment on original LinkedIn post (keeps engagement)
- [ ] Quote-tweet your own thread with a single insight
- [ ] Share to relevant LinkedIn groups (if any)
- [ ] Check Stripe for first payments
- [ ] Review YouTube analytics (CTR, average watch time)
- [ ] Reply to ALL YouTube comments

### Day 2
- [ ] LinkedIn: Post Variant C (Problem-Agitate-Solve) for second wave
- [ ] Twitter: Post Variant C thread (different from LinkedIn content)
- [ ] Reach out to engaged commenters via DM (both platforms)
- [ ] Review and fix any bugs reported
- [ ] Pin YouTube video to channel

### Week 1
- [ ] Send thank-you emails to first customers
- [ ] Gather testimonials (with permission)
- [ ] Plan Week 2 content
- [ ] Review all platform analytics:
  - [ ] YouTube: Views, CTR, watch time, subscribers gained
  - [ ] Twitter: Impressions, profile visits, followers gained
  - [ ] LinkedIn: Impressions, engagement rate, profile views

---

## Emergency Procedures

### Generation Failing

1. **Check Claude API**
   - Anthropic console: https://console.anthropic.com/
   - Verify API key valid
   - Check rate limits

2. **Check Database**
   - NeonDB dashboard
   - Verify connections not maxed

3. **Emergency Kill Switch**
   ```
   # In Vercel environment variables:
   DISABLE_GENERATION=true
   ```
   This stops all generation immediately (no redeploy needed)

### Payment Failing

1. **Check Stripe Dashboard**
   - https://dashboard.stripe.com/
   - Look for failed webhooks
   - Check API logs

2. **Verify Webhook Secret**
   - Must match between Stripe and Vercel

### Site Down

1. **Check Vercel Status**
   - https://www.vercel-status.com/

2. **Check NeonDB Status**
   - https://status.neon.tech/

3. **Rollback if Needed**
   ```bash
   # In Vercel dashboard, select previous deployment
   # Click "Promote to Production"
   ```

---

## Key URLs & Dashboards

| Service | URL | Purpose |
|---------|-----|---------|
| Vercel | https://vercel.com/dashboard | Deployment, logs |
| Stripe | https://dashboard.stripe.com/ | Payments |
| Anthropic | https://console.anthropic.com/ | Claude API usage |
| NeonDB | https://console.neon.tech/ | Database |
| Upstash | https://console.upstash.com/ | Rate limiting (if enabled) |
| Resend | https://resend.com/emails | Email logs |

---

## Launch Day Schedule Template

```
07:30 CET - Wake up, coffee
08:00 CET - Final site check (generation, payment)
08:30 CET - Copy LinkedIn post + Twitter thread, final review
09:00 CET - POST TO LINKEDIN (Go Live!)
09:05 CET - Like own post, watch for first comments
09:15 CET - POST TWITTER THREAD (7 tweets)
09:20 CET - Reply to Twitter thread with native video
09:30 CET - UPLOAD TO YOUTUBE (use youtube-metadata.md)
09:45 CET - Reply to first comments (all platforms)
10:00 CET - Check Vercel logs, any errors?
12:00 CET - Lunch break (set phone alerts for comments)
14:00 CET - Engagement check, reply to new comments (all platforms)
16:00 CET - Quote-tweet own thread with insight
18:00 CET - End of day check, metrics review (LinkedIn + Twitter + YouTube)
22:00 CET - Final check before bed
```

---

## Success Metrics (First Week)

| Metric | Target | How to Measure |
|--------|--------|----------------|
| LinkedIn impressions | 1,000+ | LinkedIn Analytics |
| LinkedIn engagement | 50+ reactions | LinkedIn post stats |
| Twitter impressions | 5,000+ | Twitter Analytics |
| Twitter engagement | 100+ (likes + retweets) | Twitter post stats |
| YouTube views | 100+ | YouTube Studio |
| YouTube watch time | 60%+ average | YouTube Studio |
| YouTube CTR | 4%+ | YouTube Studio |
| Site visits | 200+ | Vercel Analytics |
| Scenarios generated | 50+ | Database query |
| First payment | 1+ | Stripe dashboard |
| Email signups | 10+ | Contact form submissions |

---

## Post-Launch Content Calendar

| Day | Content | Platform |
|-----|---------|----------|
| Day 1 | Launch post (Variant A) + Demo video | LinkedIn, Twitter, YouTube |
| Day 2 | Quote-tweet with insight / LinkedIn comment | Twitter, LinkedIn |
| Day 3 | Problem-Agitate-Solve (Variant C) | LinkedIn, Twitter |
| Day 7 | Behind the scenes / lessons learned | LinkedIn, Twitter |
| Day 14 | Customer testimonial (if available) | LinkedIn, Twitter |
| Day 21 | Feature deep-dive (data export) | LinkedIn, Twitter, YouTube |
| Day 28 | Month 1 retrospective | LinkedIn, Twitter |

**Content Reference Files:**
- LinkedIn posts: `docs/marketing/launch/linkedin-posts.md`
- Twitter posts: `docs/marketing/launch/twitter-posts.md`
- YouTube metadata: `docs/marketing/launch/youtube-metadata.md`
- Demo video script: `docs/marketing/launch/demo-video-script.md`

---

## Critical Reminders

**NEVER post without checking:**
- [ ] No day job mentions
- [ ] No family member names
- [ ] "inspired by IBCS©" (not "IBCS-compliant")
- [ ] Correct pricing (€29-€999 lifetime)
- [ ] Realistic timing ("2-3 minutes", not "60 seconds")

**See:** `docs/marketing/CRITICAL-GUIDELINES.md`

---

**Document Version:** 1.0
**Next Review:** After launch complete
