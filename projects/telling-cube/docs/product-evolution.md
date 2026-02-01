# Product Evolution - Speed & Pre-Generation Strategy

**Date:** 2025-12-09
**Context:** Post-Phase 3 testing revealed speed issues
**Decision:** Product pivot toward pre-generated sector templates

---

## Mario's Key Insight

**"Speed shouldn't correlate with company size - only the numbers change."**

This realization led to rethinking the entire generation approach.

---

## Current State Issues

### Test Results (Dec 9, 2025)
| Scenario | Duration | Target | Issue |
|----------|----------|--------|-------|
| Bakery | 4-5 min | 90-120s | 2-3x too slow |
| Hotel | 11-12 min | 90-120s | 6-10x too slow |
| Tech Startup | 7 min | 90-120s | 3-5x too slow |

### Root Causes Identified

1. **Inconsistent Batching:**
   - Bakery: 4 API calls (quarterly)
   - Hotel: 4 API calls (quarterly) BUT taking 11 min (needs investigation)
   - Tech Startup: 12 API calls (monthly)

2. **Architectural Limitation:**
   - Sequential Claude API calls = minimum 2-3 min per scenario
   - Cannot hit 90-120s with current approach
   - Complex user data would be 10-15 minutes

3. **False Premise:**
   - Assumed complexity scales with company size
   - Reality: Same prompt structure, only numbers change
   - ALL scenarios should take ~same time

---

## Two-Phase Solution

### **Phase 1: Quick Fix (This Week)**

**Goal:** Make all 3 scenarios consistent speed (2-3 minutes)

**Actions:**
1. ✅ Standardize batching across all scenarios (quarterly = 4 calls)
2. 🔍 Investigate why Hotel takes 11 minutes (should be 2-3 min)
3. 🔧 Fix display bugs (COGS/Payroll showing €0)
4. 🔧 Fix data quality issues (Hotel missing payroll, Tech Startup invalid sales)
5. 🔧 Fix progress tracking (stuck at 10%)

**Timeline:** 1-2 days
**Owner:** James (Dev) + Winston (Architect)

**Success Criteria:**
- All 3 scenarios complete in 2-3 minutes
- Display shows correct data
- Progress tracking works
- Validation passes

---

### **Phase 2: Product Pivot - Pre-Generated Sector Templates (Next Sprint)**

**Goal:** Hit 90-120 second promise via pre-generation + customization

#### GICS Sector Templates

Pre-generate templates for 11 GICS sectors:

1. **Information Technology** (e.g., SaaS, Software, IT Services)
2. **Financials** (e.g., Banks, Insurance, Asset Management)
3. **Consumer Discretionary** (e.g., Retail, Hotels, Restaurants)
4. **Health Care** (e.g., Hospitals, Pharma, Medical Devices)
5. **Industrials** (e.g., Manufacturing, Construction, Logistics)
6. **Communication Services** (e.g., Telecom, Media, Entertainment)
7. **Consumer Staples** (e.g., Food, Beverages, Household Products)
8. **Materials** (e.g., Chemicals, Metals, Mining)
9. **Energy** (e.g., Oil & Gas, Renewable Energy)
10. **Utilities** (e.g., Electric, Water, Gas)
11. **Real Estate** (e.g., REITs, Property Management)

#### New User Flow

```
1. User selects GICS sector
   ↓
2. User picks sub-industry template
   (e.g., Consumer Discretionary → Hotel)
   ↓
3. User customizes:
   - Company name
   - Size (small/medium/large)
   - Specific focus areas
   - Optional: Upload real data points
   ↓
4. AI adapts pre-generated template
   (90-120 seconds)
   ↓
5. Results ready with customized data
```

#### Technical Architecture

**Pre-Generation (Nightly Cron):**
```typescript
// Run nightly at 2 AM
for each sector:
  for each size (small/medium/large):
    generate base template
    store in database
    cache for 30 days
```

**Runtime Customization (Fast):**
```typescript
// When user requests
1. Load pre-generated template (instant)
2. AI customizes:
   - Replace company name
   - Adjust revenue/cost scales
   - Personalize trends
   - Add user-specific events
3. Light validation (10s)
4. Return results (total: 90-120s)
```

**Benefits:**
- ✅ Hits 90-120 second promise
- ✅ Scales to complex scenarios
- ✅ Professional industry-standard classification (GICS)
- ✅ Template quality improves over time
- ✅ Lower AI costs per user

**Trade-offs:**
- Need storage for ~33 templates (11 sectors × 3 sizes)
- Initial template generation takes time
- Less "from scratch" magic (but users may not care)

---

## Current Demo Scenarios

Keep current 3 scenarios as "Quick Start" options:
- Bakery (Consumer Discretionary - Small)
- Hotel (Consumer Discretionary - Medium)
- Tech Startup (Information Technology - Small)

But add "Custom from Sector" option for power users.

---

## UX Promise Update

### Current (Broken)
> "Typically takes 90-120 seconds"

### Phase 1 (Honest)
> "Generation takes 2-3 minutes"

### Phase 2 (Achievable)
> "Select from 11 industry sectors. Customization takes 90-120 seconds."

---

## Implementation Priority

### Immediate (This Week)
1. Phase 1 quick fixes
2. Consistent 2-3 min generation
3. Fix bugs (display, data quality, progress)

### Next Sprint (Week of Dec 16)
1. Design GICS sector template structure
2. Pre-generate 3 pilot templates (Bakery, Hotel, Tech Startup)
3. Build customization UI
4. Test 90-120s customization flow

### Future (January)
1. Pre-generate all 33 templates (11 sectors × 3 sizes)
2. Launch "Custom from Sector" feature
3. Marketing: Position as "Industry-standard scenario generator"

---

## Success Metrics

### Phase 1
- [ ] All scenarios: 2-3 min consistent
- [ ] Zero display bugs
- [ ] 95%+ validation pass rate
- [ ] Brother re-validation: PASS

### Phase 2
- [ ] Customization: 90-120 seconds
- [ ] User satisfaction: >80% (speed expectations met)
- [ ] Template usage: >60% of generations use templates
- [ ] Cost reduction: 50% fewer Claude API calls

---

## Open Questions

1. **Template Refresh:** How often to regenerate templates? (Monthly? On-demand?)
2. **Customization Depth:** How much can users customize without breaking validation?
3. **Real Data Upload:** Should users be able to upload CSV to inform customization?
4. **Pricing:** Do pre-generated templates affect pricing tiers?

---

## Team Alignment

**Mario's Direction:** "Let's give Phase 1 a last try, then pivot to pre-generation"

**Consensus:**
- ✅ Winston: Phase 1 fixes are achievable (1-2 days)
- ✅ James: Can implement template system (1 sprint)
- ✅ Sally: UX for sector selection is better than "blank slate"
- ✅ Quinn: Templates easier to validate than dynamic generation

---

**Status:** Phase 1 in progress (Dec 9, 2025)
**Next Review:** After Phase 1 completion
**Long-term Strategy:** Pre-generated GICS sector templates (Phase 2)
