# Story 8.2: Label Category Mapping + Manual Override

Status: review

## Story

As a **GVS pipeline**,
I want **logic to map raw labels to governance treatment categories**,
so that **each wallet gets a clear EXCLUDE/FLAG/KEEP classification**.

## Acceptance Criteria

1. **AC-2.1:** `src/lib/labels/category-mapping.ts` — function `mapLabelToGovernance(labels: string[]): { category: LabelCategory, treatment: GovernanceTreatment }`
2. **AC-2.2:** Mapping follows the taxonomy:
   - TREASURY → EXCLUDE
   - CEX (Exchange) → EXCLUDE
   - BRIDGE → EXCLUDE
   - PROTOCOL → EXCLUDE
   - MULTISIG → FLAG
   - FUND/VC → KEEP
   - INDIVIDUAL → KEEP
   - UNKNOWN (no label) → KEEP
3. **AC-2.3:** Dual-label conflict resolution — highest restriction wins (EXCLUDE > FLAG > KEEP)
4. **AC-2.4:** `src/lib/labels/manual-overrides.json` — JSON file with manually curated labels for top known wallets (minimum 20 entries: Lido Treasury, Coinbase, Binance, Kraken, major bridges, Gnosis Safe deployer)
5. **AC-2.5:** Manual overrides take priority over eth-labels API results
6. **AC-2.6:** TypeScript types exported: `LabelCategory`, `GovernanceTreatment`, `WalletLabel` (already in types.ts from Story 8-1)

## Tasks / Subtasks

- [x] **Task 1: Category Mapping Function** (AC: 2.1, 2.2, 2.3)
  - [x] 1.1 Create `src/lib/labels/category-mapping.ts`
  - [x] 1.2 Implement `mapLabelToGovernance(labels: string[])` with keyword matching
  - [x] 1.3 Implement dual-label conflict resolution (EXCLUDE > FLAG > KEEP)

- [x] **Task 2: Manual Overrides** (AC: 2.4, 2.5)
  - [x] 2.1 Create `src/lib/labels/manual-overrides.json` with 23 entries
  - [x] 2.2 Implement `getManualOverride(address: string)` function in eth-labels-client.ts
  - [x] 2.3 Ensure manual overrides take priority in lookup flow

- [x] **Task 3: Integration with eth-labels-client** (AC: 2.5, 2.6)
  - [x] 3.1 Update `lookupWalletLabels()` to apply category mapping after API lookup
  - [x] 3.2 Update `lookupWalletLabels()` to check manual overrides first (before DB cache)
  - [x] 3.3 Ensure WalletLabelResult now has correct category + treatment from mapping

- [x] **Task 4: Verification** (all ACs)
  - [x] 4.1 TypeScript check passes (zero production errors)
  - [x] 4.2 Mapping logic: keyword matching with priority resolution
  - [x] 4.3 Conflict resolution: highest restriction wins per TREATMENT_PRIORITY
  - [x] 4.4 Manual override priority verified in lookup flow order

## Dev Notes

### Category Mapping Logic

eth-labels returns raw label strings like `["Exchange", "Coinbase"]`. Keyword matching against `LABEL_KEYWORDS` array maps to `LabelCategory`, then `CATEGORY_TREATMENT` maps to `GovernanceTreatment`.

### Treatment Priority

```
EXCLUDE (3) > FLAG (2) > KEEP (1)
```

### Manual Override: 23 entries covering

- Lido: stETH Token, Treasury, Agent (Aragon)
- CEX: Coinbase (3), Binance (2), Kraken (2)
- Bridges: Polygon, Avalanche, Optimism, Arbitrum, Wormhole
- Multisig: Gnosis Safe Master Copy + Singleton
- Fund/VC: a16z, Paradigm
- DAO Treasuries: ENS, Uniswap, Gitcoin
- Protocol: USDC (Circle)

### References

- [Source: docs/stories/epic-wallet-label-filtering.md — Story 2 ACs]
- [Source: docs/analysis/product-brief-wallet-label-filtering-2026-02-11.md — Label Taxonomy]
- [Source: src/lib/labels/types.ts — LabelCategory, GovernanceTreatment types]
- [Source: src/lib/labels/eth-labels-client.ts — Updated lookup flow]

## Dev Agent Record

### Agent Model Used

Claude Opus 4.6 (claude-opus-4-6)

### Debug Log References

- TypeScript check: zero errors in production code

### Completion Notes List

- **Task 1:** Created `category-mapping.ts` with `mapLabelToGovernance()` — 60+ keyword mappings across 8 categories. Dual-label conflict resolution using `TREATMENT_PRIORITY` (EXCLUDE:3 > FLAG:2 > KEEP:1). Also exports `getTreatmentForCategory()`.
- **Task 2:** Created `manual-overrides.json` with 23 curated entries covering Lido (3), CEX (7), Bridges (5), Multisig (2), Fund/VC (2), DAO Treasuries (3), Protocol (1). Added `getManualOverride()` in eth-labels-client.ts.
- **Task 3:** Updated `lookupWalletLabels()` with 3-tier priority: (1) manual overrides first, (2) DB cache, (3) API with `mapLabelToGovernance()` applied to raw labels before storage. Category and treatment now correctly populated in all paths.
- **Task 4:** TypeScript check passed. Mapping logic, conflict resolution, and override priority verified in code structure.

### File List

- `src/lib/labels/category-mapping.ts` — CREATED: Label→Category→Treatment mapping with keyword matching
- `src/lib/labels/manual-overrides.json` — CREATED: 23 curated wallet label overrides
- `src/lib/labels/eth-labels-client.ts` — MODIFIED: Added manual override check, category mapping integration
- `docs/stories/8-2-label-category-mapping-manual-override.md` — MODIFIED: Status → review, tasks marked complete
