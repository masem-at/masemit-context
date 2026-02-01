# Sector Financial Constraints - Technical Specification

**Author:** Winston (Architecture)
**Date:** December 2024
**Status:** Ready for Development

---

## Overview

Based on PoC-Test-2 feedback from Matthias, the generated financial data lacks business realism by sector. This spec defines sector-specific financial constraints to ensure realistic margin ranges.

---

## Problem Statement

| Scenario | Current Issue | Expected Range |
|----------|---------------|----------------|
| **Largecap Finance** | Cost/income <10%, Net margin 91% | Cost/income 40-70%, Net margin 10-30% |
| **Midcap Industrial** | Payroll 0%, Net margin 40%, Gross margin 73% | Payroll 15-30%, Net margin 5-15%, Gross margin 25-45% |
| **Startup Consumer** | ✅ Already realistic | Loss-making is appropriate |

---

## Solution: Add Financial Constraints to TierConfig

### New Interface Properties

Add to `TierConfig` in `lib/config/company-tiers.ts`:

```typescript
interface TierConfig {
  // ... existing properties ...

  // NEW: Financial constraints for realistic generation
  financialConstraints: {
    // Cost structure (as % of revenue)
    costIncomeRatio: { min: number; max: number }  // Total operating costs / Revenue
    payrollShare: { min: number; max: number }     // Payroll / Revenue
    cogsShare: { min: number; max: number }        // COGS / Revenue (gross margin inverse)

    // Margin targets (as % of revenue)
    grossMargin: { min: number; max: number }      // (Revenue - COGS) / Revenue
    netMargin: { min: number; max: number }        // Net Profit / Revenue

    // Special flags
    allowNegativeNetMargin: boolean               // Startups can have losses
  }
}
```

### Constraint Values by Sector

```typescript
// Finance / Large Cap
financialConstraints: {
  costIncomeRatio: { min: 0.40, max: 0.70 },    // 40-70% cost/income
  payrollShare: { min: 0.20, max: 0.40 },       // 20-40% of revenue
  cogsShare: { min: 0.01, max: 0.05 },          // 1-5% (fee-based, low COGS)
  grossMargin: { min: 0.95, max: 0.99 },        // 95-99% (services)
  netMargin: { min: 0.10, max: 0.30 },          // 10-30% net
  allowNegativeNetMargin: false
}

// Industrials / Mid Cap
financialConstraints: {
  costIncomeRatio: { min: 0.75, max: 0.90 },    // 75-90% cost/income
  payrollShare: { min: 0.15, max: 0.30 },       // 15-30% of revenue
  cogsShare: { min: 0.40, max: 0.60 },          // 40-60% COGS (manufacturing)
  grossMargin: { min: 0.40, max: 0.60 },        // 40-60% gross margin
  netMargin: { min: 0.05, max: 0.15 },          // 5-15% net
  allowNegativeNetMargin: false
}

// Consumer Staples / Startup
financialConstraints: {
  costIncomeRatio: { min: 1.00, max: 2.00 },    // 100-200% (burning cash)
  payrollShare: { min: 0.20, max: 0.35 },       // 20-35% of revenue
  cogsShare: { min: 0.35, max: 0.50 },          // 35-50% COGS
  grossMargin: { min: 0.50, max: 0.65 },        // 50-65% gross margin
  netMargin: { min: -1.00, max: 0.05 },         // -100% to +5% (losses OK)
  allowNegativeNetMargin: true
}
```

---

## Implementation Changes

### 1. Update `lib/config/company-tiers.ts`

Add `financialConstraints` to each tier config with the values above.

### 2. Update Stage 2 Prompt (`lib/prompts/tier-scenario.ts`)

Modify `getTierStage2Prompt()` to include constraint instructions:

```typescript
// Add to prompt:
**CRITICAL FINANCIAL CONSTRAINTS for ${config.sector}:**
Your generated events MUST produce these financial ratios:
- Cost/Income Ratio: ${config.financialConstraints.costIncomeRatio.min * 100}% - ${config.financialConstraints.costIncomeRatio.max * 100}%
- Payroll as % of Revenue: ${config.financialConstraints.payrollShare.min * 100}% - ${config.financialConstraints.payrollShare.max * 100}%
- COGS as % of Revenue: ${config.financialConstraints.cogsShare.min * 100}% - ${config.financialConstraints.cogsShare.max * 100}%
- Gross Margin: ${config.financialConstraints.grossMargin.min * 100}% - ${config.financialConstraints.grossMargin.max * 100}%
- Net Margin: ${config.financialConstraints.netMargin.min * 100}% - ${config.financialConstraints.netMargin.max * 100}%

These are HARD CONSTRAINTS. Generate appropriate volumes of:
- SALE events (revenue)
- PROCUREMENT events (COGS)
- PAYROLL events (ensure payroll costs meet the range)
- OPEX events (other operating expenses)
```

### 3. Update Stage 3 Prompts

Apply same constraints to quarterly/monthly generation prompts.

---

## Files to Modify

1. `lib/config/company-tiers.ts` - Add financialConstraints to TierConfig interface and each tier
2. `lib/prompts/tier-scenario.ts` - Add constraint instructions to Stage 2 and Stage 3 prompts

---

## Testing

After implementation, generate new scenarios for each tier and verify:

| Metric | Largecap | Midcap | Startup |
|--------|----------|--------|---------|
| Cost/Income | 40-70% | 75-90% | 100-200% |
| Payroll/Revenue | 20-40% | 15-30% | 20-35% |
| Gross Margin | 95-99% | 40-60% | 50-65% |
| Net Margin | 10-30% | 5-15% | -100% to +5% |

---

## Notes

- The AI may not hit exact ranges every time, but should be much closer than current output
- Consider adding validation post-generation to flag scenarios outside constraints
- Future enhancement: Allow users to customize constraints per scenario
