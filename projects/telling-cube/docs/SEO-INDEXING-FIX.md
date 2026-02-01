# SEO Indexing Issue - Root Cause & Fix

**Date:** 2025-12-21
**Status:** CRITICAL - Blocking Google indexing

## Problem Summary

Google Search Console reports:
- 5 pages excluded by "noindex" tag
- 1 page "Crawled - currently not indexed"
- Sitemap pointing to wrong domain (Vercel preview URL)

## Root Causes

### 1. Launch Proxy Still Active on Production
**Evidence:**
- `middleware.ts` checks `LAUNCH_PROXY_ENABLED` environment variable
- When true, ALL requests to tellingcube.com are rewritten to `/launch-proxy`
- `/app/launch-proxy/layout.tsx` has `robots: { index: false, follow: false }`

**Impact:** Every page Google crawls shows "noindex", preventing indexing

### 2. Wrong Base URL Configuration
**Evidence:**
- `robots.ts` and `sitemap.ts` fall back to `VERCEL_URL` when `NEXT_PUBLIC_BASE_URL` not set
- Current robots.txt shows: `Sitemap: https://telling-cube-njhb7e2v5-masemit-projects.vercel.app/sitemap.xml`

**Impact:** Google sees wrong sitemap URL, can't properly crawl site structure

### 3. Contact Page Missing from Sitemap
**Evidence:**
- `app/sitemap.ts` only included: home, /privacy, /terms
- Missing: /contact

**Impact:** Contact page not being discovered by Google crawler

## The Fix

### Code Changes (Already Applied)
✅ Added /contact to sitemap.ts with priority 0.8

### Vercel Environment Variable Changes (YOU MUST DO THIS)

Go to Vercel Dashboard → tellingcube → Settings → Environment Variables → Production

#### Critical: Disable Launch Proxy
**Variable:** `LAUNCH_PROXY_ENABLED`
**Action:** Either:
- Set value to `false`, OR
- Delete the variable entirely

**Why:** This removes the noindex middleware that's blocking all pages

#### Critical: Set Production Base URL
**Variable:** `NEXT_PUBLIC_BASE_URL`
**Current value:** Unknown (likely not set or wrong)
**New value:** `https://tellingcube.com`

**Why:** Ensures robots.txt and sitemap.xml use correct production domain

### After Making Changes

1. **Trigger Redeployment**
   - Vercel will auto-deploy when you update environment variables
   - Or manually redeploy from Vercel dashboard

2. **Verify the Fix**
   ```bash
   # Check robots.txt shows correct sitemap
   curl https://tellingcube.com/robots.txt
   # Should show: Sitemap: https://tellingcube.com/sitemap.xml

   # Check sitemap includes contact page
   curl https://tellingcube.com/sitemap.xml
   # Should include /contact entry

   # Check homepage doesn't have noindex
   curl -s https://tellingcube.com/ | grep -i "noindex"
   # Should return nothing (no matches)
   ```

3. **Request Re-indexing in Google Search Console**
   - Go to URL Inspection tool
   - Test each affected URL (home, contact, privacy, terms)
   - Click "Request Indexing" for each
   - Google will recrawl within 1-2 days

## Technical Details

### Middleware Flow (When Launch Proxy Enabled)
```
Request to tellingcube.com/
  ↓
middleware.ts checks LAUNCH_PROXY_ENABLED=true
  ↓
Rewrites to /launch-proxy
  ↓
Serves page with robots: { index: false }
  ↓
Google sees noindex, skips indexing
```

### Correct Flow (After Fix)
```
Request to tellingcube.com/
  ↓
middleware.ts checks LAUNCH_PROXY_ENABLED=false
  ↓
Serves actual homepage from app/page.tsx
  ↓
Uses robots config from app/layout.tsx: { index: true }
  ↓
Google indexes normally
```

### Files Involved
- `middleware.ts` - Launch proxy routing logic
- `app/launch-proxy/layout.tsx` - Page with noindex
- `app/layout.tsx` - Main layout with index: true
- `app/robots.ts` - Generates robots.txt
- `app/sitemap.ts` - Generates sitemap.xml

## Verification Checklist

After deployment:
- [ ] `LAUNCH_PROXY_ENABLED` is set to `false` or deleted in Vercel Production
- [ ] `NEXT_PUBLIC_BASE_URL` is set to `https://tellingcube.com` in Vercel Production
- [ ] robots.txt shows correct sitemap URL (tellingcube.com, not Vercel preview)
- [ ] sitemap.xml includes /contact page
- [ ] Homepage source doesn't contain `<meta name="robots" content="noindex">`
- [ ] Google Search Console URL Inspection shows "Indexing allowed"
- [ ] Re-indexing requested for all affected URLs

## Timeline to Resolution

1. **Immediate** (5 min): Update Vercel environment variables
2. **5-10 min**: Vercel auto-deployment completes
3. **1 hour**: DNS/CDN propagation
4. **1-2 days**: Google recrawls after re-index request
5. **3-7 days**: Pages appear in Google Search results

## Prevention

**Future deployments should:**
1. Always verify `LAUNCH_PROXY_ENABLED=false` for production
2. Ensure `NEXT_PUBLIC_BASE_URL` is set to correct production domain
3. Test robots.txt and sitemap.xml URLs before launch
4. Use Google Search Console's URL Inspection tool to verify indexing status
