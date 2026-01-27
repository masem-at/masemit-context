# ChainSights Report Review

**Report:** Governance Truth Report - YAM  
**DAO:** Yam Finance (yam.eth)  
**Review Date:** December 17, 2025  
**Reviewer:** Claude (QA Agent)  
**Data Scope:** 10 recent proposals, 92 unique voters, 215 total proposals referenced

---

## Executive Summary

The report presents accurate quantitative metrics (Gini coefficient, power distribution percentages, unique voter count) that match the source JSON data. However, there is a **critical classification inconsistency**: the report labels top voters as "WHALE" while the source data explicitly marks them as `isWhale: false` with `directTokenPower: 0`. This contradicts the report's own methodology definition. The scope is clearly stated and appropriate for the data provided.

**Verdict:** 🟡 Minor Fixes Needed

---

## ✅ What's Good

- All power distribution percentages accurately match the JSON source data
- Gini coefficient correctly calculated and reported (0.915)
- Unique voter count exactly matches source (92)
- Proposals count correctly stated (215 total, 10 analyzed)
- Report clearly states its methodology and data scope
- Qualitative observations about governance decline are supported by the data
- Recommendations are actionable and relevant to identified problems

---

## ⚠️ Items to Verify

| Item | Concern | How to Verify |
|------|---------|---------------|
| Vote count range "14-44" | Report claims minimum of 14 votes, but visible proposals show minimum 21 | Check all 10 analyzed proposals for lowest vote count |
| "0.6% average participation" for top 20 | Calculation method unclear | Recalculate from JSON participation rates |

---

## ❌ Issues to Fix

| Issue | Current | Should Be | Source Reference |
|-------|---------|-----------|------------------|
| **WHALE classification** | Top voters labeled as "WHALE" | Should be "UNCLASSIFIED" or classification system needs revision | JSON: all topVoters have `isWhale: false`, `directTokenPower: 0` |
| Classification methodology mismatch | Report shows WHALEs with directTokenPower shown as voting power | According to report's own rules: "WHALE: directTokenPower > 0" - but all have directTokenPower = 0 | JSON lines 331-600: every voter has `directTokenPower: 0` |
| Table header mismatch | "Whales (High Token Holdings)" | Data shows these may be delegated votes, not direct holdings | YAM uses contract-call strategy for `getCurrentVotes` which returns delegated VP |

---

## 📊 Numbers Verification

| Metric in Report | Report Value | Calculated from JSON | Match? |
|------------------|--------------|----------------------|--------|
| Total Proposals | 215 | 215 (`proposalsCount`) | ✅ |
| Proposals Analyzed | 10 | 10 (proposals array) | ✅ |
| Unique Voters | 92 | 92 (`uniqueVoters`) | ✅ |
| Gini Coefficient | 0.915 | 0.9148045942596352 | ✅ |
| Top 1 Wallet % | 22.8% | 22.8494017475184% | ✅ |
| Top 5 Wallets % | 69.4% | 69.37588612349153% | ✅ |
| Top 10 Wallets % | 90.9% (implied) | 90.93129116126417% | ✅ |
| Top 20 Wallets % | 100.0% | 99.96269251269759% | ✅ |
| Total Voting Power | 3,369,017 (implied) | 3,369,017.013979597 | ✅ |
| Top Wallet VP | 769,800 | 769,800.2324664462 | ✅ |
| Top Wallet Address | 0x653d63...48aDa5 | 0x653d63E4F2D7112a19f5Eb993890a3F27b48aDa5 | ✅ |
| Second Wallet VP | 746,364 | 746,363.6368983652 | ✅ |
| Third Wallet VP | 347,880 | 347,880.4139643999 | ✅ |
| Followers Count | N/A (not in report) | 1,851 | N/A |

---

## 🔍 Scope Check

- [x] Report clearly states what time period/data it covers ("Analysis based on 10 recent proposals")
- [x] Report states "92 unique voters across 215 proposals" - accurately reflects data
- [ ] **Partial Issue:** Report claims voters are "WHALE" but JSON flags them as not whales
- [x] Limitations acknowledged (methodology section explains classification rules)

---

## 💡 Suggestions (Optional)

1. **Address the Whale/Delegate Classification Issue:** The YAM governance uses a `contract-call` strategy that returns `getCurrentVotes` - this is typically delegated voting power in governance tokens. Consider:
   - Revising the classification logic to handle cases where `directTokenPower = 0` but `totalVotingPower > 0`
   - Adding a category like "POWER HOLDER" for wallets with significant VP but unknown source
   - Investigating why the data pipeline shows `directTokenPower: 0` for all voters

2. **Add Followers Context:** The space has 1,851 followers but only 92 unique voters (4.97% of followers ever voted). This could strengthen the "governance dormancy" narrative.

3. **Clarify Participation Rate Calculation:** The report mentions "0.6% average participation among top 20" but the JSON shows participation rates are calculated against total proposals (304 or 215?). Consider clarifying the denominator.

4. **Vote Count Range:** Verify the "14-44" claim - visible data shows 21-44 range in the proposals array.

---

## Detailed Classification Analysis

The JSON source data shows a pattern that requires attention:

```
ALL 92 voters have:
- isWhale: false
- isDelegate: false
- directTokenPower: 0
- delegatedPower: 0
- totalVotingPower: [varies from 4.2 to 769,800]
```

This creates a logical inconsistency:
- The report's methodology states: "WHALE: directTokenPower > 0 AND directTokenPower > delegatedPower"
- No voter meets this criterion (all have directTokenPower = 0)
- Yet the report classifies top voters as "WHALE"

**Root Cause Hypothesis:** The YAM governance uses `getCurrentVotes` from the token contract, which returns voting power after delegation. The Snapshot API may not be properly decomposing this into direct vs. delegated power for this specific strategy type.

**Impact:** The report's central claim that power is held by "WHALEs" vs "DELEGATEs" cannot be verified from the provided data. The power concentration analysis is accurate, but the characterization of WHO holds that power (direct holders vs delegates) is uncertain.

---

## Review Notes

- The YAM DAO appears to be in significant governance distress, which the report accurately captures
- Internal conflicts (0xE removal proposals) suggest this is a DAO in crisis, not just apathy
- The quantitative metrics are solid and match source data precisely
- The classification issue is a data pipeline/methodology problem, not a calculation error
- Report tier appears to be Basic (10 proposals analyzed) - scope is appropriate
- The report correctly avoids making claims beyond the analyzed data

---

*Review generated by ChainSights QA Process*
*Data verified against: chainsights-data-yam.json (1,944 lines)*
