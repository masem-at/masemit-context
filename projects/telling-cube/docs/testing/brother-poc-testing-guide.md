# tellingCube PoC Testing Guide
## For Brother (Controller Validation)

**Tester:** Mario's Brother (Controller, 250-300 person company experience)
**Environment:** Production (Vercel deployment)
**Test URL:** [Mario will provide the Vercel URL: https://telling-cube-XXXXX.vercel.app]
**Duration:** 30-45 minutes
**Goal:** Validate IBCS compliance and data realism for demo purposes

> **Note:** The custom domain (tellingcube.com) is not yet connected. You'll be testing on the Vercel subdomain, but all functionality is identical to the final production site.

---

## 📋 What You're Testing

You're testing whether **tellingCube** generates realistic business data that:
1. **Looks professional** (IBCS-compliant charts)
2. **Makes business sense** (realistic numbers for a bakery/hotel/startup)
3. **Is mathematically consistent** (Finance numbers match Sales numbers)

Your expertise as a controller is critical - you'll spot unrealistic patterns that others might miss.

---

## 🚀 Step-by-Step Test Plan

### **Test 1: Generate Bakery Scenario** (10 minutes)

#### Step 1.1: Open the Website
1. Go to: **[The URL Mario sent you - should be https://telling-cube-XXXXX.vercel.app]**
   - **Important:** Use the EXACT URL Mario provided (Vercel subdomain)
   - If you see a password prompt, enter the password Mario gave you
2. You should see:
   - Large heading: "Realistic business data in minutes, not hours"
   - Three blue buttons for different scenarios
   - Pricing section below (scroll down)

#### Step 1.2: Start Generation
1. Click the **"Generate Bakery Scenario"** button
2. You'll be redirected to a loading page
3. Wait 30-90 seconds (typical: 60 seconds)
4. Watch the status messages:
   - "Generating master data..."
   - "Generating business events..."
   - "Validating consistency..."

**⏱️ Note the time:** How long did generation take? ___________ seconds

#### Step 1.3: Review the Results Page

**Desktop View:**
You should see two dashboards side-by-side:
- **Left:** Sales View
- **Right:** Finance View

**Mobile/Tablet View:**
You should see tabs at the top to switch between Sales and Finance.

**✅ Check:** Is the layout clear and professional? (Yes / No / Notes)

---

### **Test 2: Validate Finance View (IBCS Compliance)** (15 minutes)

This is your expertise area! Check if the Finance dashboard follows IBCS standards.

#### Step 2.1: Visual Design Check

**Charts should be:**
- ✅ **Grayscale colors only** (black, dark gray, light gray - NO blue/green)
- ✅ **No vertical grid lines** (horizontal lines only)
- ✅ **Clean axes** (no heavy borders)
- ✅ **White background**
- ✅ **Message-driven headlines** (e.g., "Revenue grew 15% in Q2" NOT just "Revenue")

**Rate the IBCS compliance:** (1-5, where 5 = perfect)
- Visual design: _____ / 5
- Chart clarity: _____ / 5
- Professional appearance: _____ / 5

**Notes/Issues:**
```
[Write any IBCS violations or concerns here]



```

#### Step 2.2: Data Realism Check

Look at the **P&L Summary** (top section of Finance View):

**Check these numbers make sense for a bakery:**
- ✅ Revenue: Is it realistic for 1 year? (€300K-€800K expected)
- ✅ Payroll: Does it match ~10 employees? (€150K-€250K expected)
- ✅ COGS: Is it 40-50% of revenue? (bakery margin check)
- ✅ Operating Profit: Is there a reasonable margin? (10-20% expected)

**Rate the realism:** (1-5, where 5 = very realistic)
- Revenue levels: _____ / 5
- Cost structure: _____ / 5
- Profit margins: _____ / 5

**Red flags to watch for:**
- ❌ Revenue jumps 500% in one month (unrealistic)
- ❌ Zero payroll for multiple months (impossible)
- ❌ COGS > Revenue (business would be bankrupt)
- ❌ Negative revenue (impossible)

**Any red flags found?** (Yes / No - describe below)
```
[Describe any unrealistic patterns]



```

#### Step 2.3: Monthly Cash Flow Check

Scroll to the **Monthly Cash Flow** chart:

**Check:**
- ✅ Cash in and cash out vary month-to-month (not flat line)
- ✅ Cash in ≈ Revenue (should be similar)
- ✅ Cash out includes payroll + COGS + rent
- ✅ Some months have positive net cash, some negative (realistic variability)

**Rate the cash flow realism:** _____ / 5

**Notes:**
```
[Any cash flow concerns]


```

---

### **Test 3: Validate Multi-View Consistency** (10 minutes)

This is tellingCube's **key differentiator** - Finance and Sales views should reconcile perfectly.

#### Step 3.1: Check the Consistency Badge

At the top of the results page, look for:
- ✅ Green badge: "Multi-view consistency verified"
- ❌ Yellow/red badge: "Validation issues detected"

**Badge status:** ✅ Green / ❌ Warning / ❌ Error

If warning/error, screenshot it!

#### Step 3.2: Manual Reconciliation Check

**Compare Sales View (left) vs Finance View (right):**

1. **Revenue Reconciliation:**
   - Sales View: Total revenue = €_________ (find in "Total Revenue" metric)
   - Finance View: P&L revenue = €_________ (find in P&L Summary)
   - **Do they match?** ✅ Yes / ❌ No (difference: €_______)

2. **Monthly Trend Check:**
   - Sales View: Look at "Revenue by Month" chart
   - Finance View: Look at "Monthly Cash Flow" (cash in)
   - **Do the monthly patterns look similar?** ✅ Yes / ❌ No

**Consistency rating:** _____ / 5

**Notes:**
```
[Any inconsistencies found]


```

---

### **Test 4: Generate Hotel Scenario** (10 minutes)

Let's test a different industry to see if the AI adapts correctly.

#### Step 4.1: Generate Hotel
1. Click the **tellingCube logo** (top-left) to return home
2. Click **"Generate Hotel Scenario"**
3. Wait 30-90 seconds

**⏱️ Generation time:** ___________ seconds

#### Step 4.2: Quick Data Check (Hotel-Specific)

**Finance View - Check for hotel-realistic numbers:**
- ✅ Revenue: €500K-€2M (hotels have higher revenue than bakeries)
- ✅ COGS: Lower % than bakery (hotels have lower material costs)
- ✅ Payroll: Higher (hotels need more staff)

**Does the data feel like a hotel business?** ✅ Yes / ❌ No

**Industry-specific check:**
- Switch to **Sales View** (left panel or tab)
- Look at **"Top Customers"** table
- **Are there customer names?** (hotels have guests/booking platforms)

**Notes:**
```
[Hotel scenario observations]


```

---

### **Test 5: Mobile Responsiveness** (5 minutes)

If you have a smartphone or tablet, test mobile:

#### Step 5.1: Open on Mobile
1. Open **[the Vercel URL Mario sent you]** on your phone
2. Generate a scenario (Bakery or Hotel)
3. View the results

**Mobile-specific checks:**
- ✅ Tabs at top to switch Sales ↔ Finance (not side-by-side)
- ✅ Charts are readable (not cut off)
- ✅ Tables scroll horizontally if needed
- ✅ Text is large enough to read

**Mobile experience rating:** _____ / 5

**Issues:**
```
[Mobile-specific problems]


```

---

## 📊 Final Assessment

### Overall Scores

| Category | Score (1-5) | Weight | Notes |
|----------|-------------|--------|-------|
| **IBCS Compliance** | _____ / 5 | HIGH | Charts, colors, design |
| **Data Realism** | _____ / 5 | HIGH | Numbers make business sense |
| **Consistency** | _____ / 5 | CRITICAL | Sales ↔ Finance reconciliation |
| **Professional Appearance** | _____ / 5 | MEDIUM | Overall polish |
| **Usability** | _____ / 5 | MEDIUM | Easy to use? |

### Go/No-Go Decision

**Would you use this for demo purposes with clients?**
- ✅ **YES** - Ready for demos, professional quality
- ⚠️ **YES with changes** - Usable, but needs fixes (list below)
- ❌ **NO** - Not realistic enough yet

**If "YES with changes", what needs fixing?**
```
Priority 1 (must fix):
1.
2.

Priority 2 (nice to have):
1.
2.
```

### Final Comments

**What impressed you most?**
```


```

**What surprised you (good or bad)?**
```


```

**Any controller-specific concerns?**
```


```

---

## 📧 Submit Your Feedback

**Option 1: Email**
Send this completed guide to: mario@masemit.at

**Option 2: Phone Call**
Call Mario and discuss findings (bring screenshots if issues found)

**Option 3: WhatsApp/Signal**
Send screenshots + voice message summary

---

## 🆘 Troubleshooting

### Generation Takes >2 Minutes
- **Normal:** First generation can be slow (server cold start)
- **Action:** Wait up to 5 minutes, then refresh and try again

### "Validation Failed" Error
- **Meaning:** The AI generated inconsistent data
- **Action:** Click browser back, generate again (should work 95% of the time)

### Charts Look Blue/Green Instead of Grayscale
- **Issue:** IBCS styling not applied correctly
- **Action:** Screenshot and send to Mario (this is a bug!)

### Can't See Results on Mobile
- **Check:** Are you scrolling? Results are below the consistency badge
- **Check:** Did generation finish? Look for the green badge

### Website Won't Load
- **Check:** Is your internet working? Try google.com
- **Backup:** Use mobile data instead of WiFi

---

## ✅ Testing Checklist (Quick Reference)

Before you start:
- [ ] Computer/phone with internet access
- [ ] 30-45 minutes of uninterrupted time
- [ ] Pen/paper for notes (or type directly in this doc)

During testing:
- [ ] Test 1: Bakery scenario generated successfully
- [ ] Test 2: Finance view IBCS compliance checked
- [ ] Test 3: Consistency verified (Sales ↔ Finance)
- [ ] Test 4: Hotel scenario tested
- [ ] Test 5: Mobile tested (if available)

After testing:
- [ ] All scores filled in
- [ ] Go/No-Go decision made
- [ ] Feedback sent to Mario

---

**Thank you for your expert validation! Your controller perspective is invaluable.** 🙏

**Estimated completion time:** 30-45 minutes
**Difficulty:** Easy (just click, observe, and rate)
**Impact:** Critical for product launch decision

---

**Version:** 1.0
**Last Updated:** 2025-11-21
**Contact:** Mario (mario@masemit.at)