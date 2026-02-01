# Vercel Access Setup for Brother's Testing
## Quick Guide for Mario

**Goal:** Give your brother access to test tellingCube on Vercel subdomain

---

## ✅ **Option 1: Public URL (EASIEST - 2 minutes)**

### Step 1: Get Your Production URL

1. **Go to Vercel Dashboard:**
   - Open: https://vercel.com/dashboard
   - Login if needed

2. **Find Your Project:**
   - Look for `telling-cube` in the projects list
   - Click on it

3. **Copy the Production URL:**
   - Look at the top of the project page
   - You'll see a URL like:
     - `https://telling-cube.vercel.app` OR
     - `https://telling-cube-[username].vercel.app` OR
     - `https://telling-cube-git-main-[username].vercel.app`
   - Click the **"Visit"** button (top-right) to copy it

4. **Test It Yourself:**
   - Open the URL in an **incognito/private browser window**
   - If it loads WITHOUT asking for a password → ✅ You're good!
   - If it asks for password → Go to Option 2 below

### Step 2: Send URL to Brother

**Email/WhatsApp template:**

```
Hey [Brother's name],

Can you test tellingCube for me? Here's everything you need:

📱 Test URL: [paste your Vercel URL here]

📄 Testing Guide: [attach brother-poc-testing-guide.md as PDF]

⏱️ Time needed: 30-45 minutes

Just open the URL, generate a Bakery scenario, and fill out the ratings in the guide. Let me know if you hit any issues!

Thanks! 🙏
Mario
```

**That's it!** If the URL is public (no password), you're done. ✅

---

## 🔐 **Option 2: URL is Password Protected (5 minutes)**

If the URL asked for a password in Step 1.4 above:

### Step 1: Check Password Protection Settings

1. **Vercel Dashboard** → Your Project → **Settings** (left sidebar)
2. Click **"Deployment Protection"**
3. You'll see one of these options:
   - ✅ **"Protection is disabled"** → URL is already public, no action needed
   - ⚠️ **"Password Protection" is enabled** → Continue below

### Step 2: Disable Password Protection (Recommended)

1. Toggle **"Password Protection"** to OFF
2. Click **"Save"**
3. Test the URL again in incognito mode
4. Now it should load without password ✅

**OR** if you want to keep password protection:

### Step 2 (Alternative): Share the Password

1. Copy the **"Protection Password"** from the settings page
2. Send it to your brother along with the URL

**Updated email template:**

```
Hey [Brother],

Test URL: [Vercel URL]
Password: [paste protection password here]

When you open the URL, it'll ask for a password - just paste the one above.

Testing guide attached. Thanks!
Mario
```

---

## 🚀 **Quick Verification Checklist**

Before sending to your brother:

- [ ] You have the Vercel production URL
- [ ] You tested it in incognito mode (loads without errors)
- [ ] If password-protected, you have the password
- [ ] Testing guide PDF is ready (docs/testing/brother-poc-testing-guide.md)
- [ ] Brother knows it's the Vercel subdomain (not tellingcube.com yet)

---

## 📧 **Full Email Template (Copy-Paste)**

```
Subject: tellingCube Testing - Need Your Expert Opinion

Hi [Brother's name],

I need your controller expertise to validate tellingCube before launching!

🔗 **Test URL:** [paste Vercel URL]
[If password-protected: 🔑 Password: XXXXX]

📄 **Testing Guide:** Attached (30-45 min, step-by-step)

**What I need:**
- Test generating 2 scenarios (Bakery + Hotel)
- Check IBCS compliance (charts, colors, layout)
- Verify data realism (do the numbers make sense for a bakery?)
- Rate everything on the scorecard in the guide

**Note:** This is the Vercel test URL (tellingcube.com domain not connected yet), but all functionality is identical to final production.

**Timeline:** Can you test by [date]? Just reply with your scores and Go/No-Go decision.

Thanks so much! Your feedback is critical for launch.

Mario
```

---

## 🆘 **Troubleshooting**

### "I can't find my Vercel project"

**Solution:**
1. Go to https://vercel.com/dashboard
2. Check the team/account dropdown (top-left)
3. Make sure you're in the correct team
4. Projects are listed on the dashboard homepage

### "The URL shows 404 Not Found"

**Possible reasons:**
- The project hasn't been deployed yet
- You're using the wrong URL
- Deployment failed

**Solution:**
1. Go to Vercel Dashboard → Your Project → **"Deployments"** tab
2. Look for the most recent deployment with status **"Ready"**
3. Click on it
4. Click **"Visit"** to get the correct URL

### "The URL shows an error page"

**Solution:**
1. Check if the deployment succeeded:
   - Dashboard → Project → Deployments
   - Latest deployment should say "Ready" (green)
2. If deployment failed, check build logs
3. Might need to redeploy (click "Redeploy" on the failed deployment)

### "I want to test locally first"

**Not recommended for brother's test** - he needs to test the actual deployed version.

But if YOU want to test locally:
```bash
pnpm dev
# Open http://localhost:3000
```

---

## ✅ **DONE!**

Once you send the URL + guide to your brother, you're all set. Just wait for his feedback!

**Next step:** After he tests → Call another Lagerfeuer to review his findings.

---

**Version:** 1.0
**Created:** 2025-11-21
**For:** Mario (internal guide)