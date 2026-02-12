# Epic: Wallet Label Filtering — Data Quality Improvement

**Source:** `docs/analysis/product-brief-wallet-label-filtering-2026-02-11.md`
**Origin:** Jenya_K (Lido DAO Operations Workstream) community feedback
**Decision:** Approved 2026-02-11 (see `project_context.md`)
**Status:** Implemented (commit `ab9f4be`, reviewed + fixes `bd1908f+`)

---

## Story 1: Database Schema + eth-labels API Client

**As a** GVS pipeline
**I need** a wallet label cache and an API client for eth-labels
**So that** wallet labels are stored locally and lookups are fast

### Acceptance Criteria

- [x] AC-1.1: `wallet_labels` table created via Drizzle migration with fields: `id`, `address` (unique, varchar 42), `label_source`, `nametag`, `label_category`, `governance_treatment` (EXCLUDE/FLAG/KEEP), `labels` (JSONB), `confidence`, `first_seen_at`, `last_verified_at`
- [x] AC-1.2: Drizzle schema in `src/lib/db/schema.ts` with indexes on `address` and `label_category`
- [x] AC-1.3: `src/lib/labels/eth-labels-client.ts` — async function `lookupWalletLabel(address: string)` that calls eth-labels API
- [x] AC-1.4: Rate limiting — max 5 requests/second to eth-labels API (batch size 5, 1s delay)
- [x] AC-1.5: Error handling — API timeout (5s), 404 (return null), 429 (backoff + retry once)
- [x] AC-1.6: `lookupWalletLabels(addresses: string[])` batch function that checks cache first, only looks up uncached addresses
- [x] AC-1.7: Results stored in `wallet_labels` table after each lookup

### Technical Notes

- eth-labels does NOT have a batch endpoint — concurrent batches of 5 with rate limiting
- Address format: lowercase 0x-prefixed (normalize before lookup + storage)
- Circuit breaker: stops API calls after 5 consecutive failures, fills remaining as UNKNOWN/KEEP
- Serialization lock prevents duplicate concurrent batch lookups

### File List

- `src/lib/labels/eth-labels-client.ts` — API client + cache + batch lookup
- `src/lib/db/schema.ts` — `walletLabels` table definition (line 934)
- `drizzle/0020_greedy_genesis.sql` — Migration SQL

---

## Story 2: Label Category Mapping + Manual Override

**As a** GVS pipeline
**I need** logic to map raw labels to governance treatment categories
**So that** each wallet gets a clear EXCLUDE/FLAG/KEEP classification

### Acceptance Criteria

- [x] AC-2.1: `src/lib/labels/category-mapping.ts` — function `mapLabelToGovernance(labels: string[]): { category: LabelCategory, treatment: GovernanceTreatment }`
- [x] AC-2.2: Mapping follows the taxonomy from the product brief:
  - TREASURY → EXCLUDE
  - CEX (Exchange) → EXCLUDE
  - BRIDGE → EXCLUDE
  - PROTOCOL → EXCLUDE
  - MULTISIG → FLAG
  - FUND/VC → KEEP
  - INDIVIDUAL → KEEP
  - UNKNOWN (no label) → KEEP
- [x] AC-2.3: Dual-label conflict resolution — highest restriction wins (EXCLUDE > FLAG > KEEP)
- [x] AC-2.4: `src/lib/labels/manual-overrides.json` — JSON file with 22 manually curated entries (Lido Treasury, Coinbase, Binance, Kraken, major bridges, Gnosis Safe)
- [x] AC-2.5: Manual overrides take priority over eth-labels API results
- [x] AC-2.6: TypeScript types exported: `LabelCategory`, `GovernanceTreatment`, `WalletLabel`

### Technical Notes

- Human activity keywords (delegate, governance, voter, staker, contributor, council, steward) override PROTOCOL/CEX classification → KEEP
- PROTOCOL keywords narrowed to contract-specific labels (router, factory, deployer) to avoid false-excluding DAO delegates
- MULTISIG 'safe' keyword narrowed to 'gnosis safe', 'safe: ', 'safe wallet' to avoid matching "SafeDAO"

### File List

- `src/lib/labels/category-mapping.ts` — Keyword mapping + conflict resolution
- `src/lib/labels/manual-overrides.json` — 22 curated entries
- `src/lib/labels/types.ts` — Type definitions

---

## Story 3: GVS Pipeline Integration (Step 1.5)

**As a** GVS scoring pipeline
**I need** to filter/flag labeled wallets before calculating scores
**So that** HPR, DEI, PDI, GPI reflect human governance participation only

### Acceptance Criteria

- [x] AC-3.1: `filterExcludedVotes(votes: SnapshotVote[])` in `src/lib/labels/enrichment.ts` — filters EXCLUDE wallets, tracks FLAG wallets, logs breakdown (note: function name differs from spec — takes votes not voters)
- [x] AC-3.2: Pipeline integration point: called **before** WHALE/DELEGATE classification in HPR, DEI, PDI, GPI and in `fetchGovernanceData()`
- [x] AC-3.3: Wallets with `treatment: EXCLUDE` are removed from:
  - Unique voter count (HPR)
  - Power distribution calculation (Gini, Nakamoto, Top-N%)
  - Grassroots participation metrics (GPI)
- [x] AC-3.4: Wallets with `treatment: FLAG` are kept in calculations but tagged (for future reporting)
- [x] AC-3.5: Wallets with `treatment: KEEP` are unchanged
- [x] AC-3.6: Excluded wallet count logged: `[Labels] Excluded X wallets (Y treasury, Z exchange, W bridge/protocol)`
- [x] AC-3.7: `PowerDistribution` result includes new field: `excludedWallets: number`

### Technical Notes

- `setSkipLabelFiltering(skip)` global bypass for validation scripts
- Filtering called in 5 locations: hpr.ts, dei.ts, pdi.ts, gpi.ts, client.ts (fetchGovernanceData)

### File List

- `src/lib/labels/enrichment.ts` — Filter function
- `src/lib/gvs/hpr.ts` — Added filterExcludedVotes call
- `src/lib/gvs/dei.ts` — Added filterExcludedVotes call
- `src/lib/gvs/pdi.ts` — Added filterExcludedVotes call
- `src/lib/gvs/gpi.ts` — Added filterExcludedVotes call
- `src/lib/snapshot/client.ts` — Added filterExcludedVotes in fetchGovernanceData
- `src/lib/snapshot/types.ts` — Added `excludedWallets` to PowerDistribution

---

## Story 4: Report + Methodology Transparency

**As a** report reader
**I need** to know that wallet filtering was applied
**So that** I understand the methodology and trust the scores

### Acceptance Criteria

- [x] AC-4.1: PDF report methodology section updated with transparency note
- [x] AC-4.2: Excluded wallet count included via `draft.methodology` text (generated by AI analysis)
- [x] AC-4.3: `/methodology` page updated with "Data Quality: Wallet Filtering" section
- [x] AC-4.4: AI analysis prompt updated to mention that data is pre-filtered

### File List

- `src/lib/pdf/report-template.tsx` — Wallet filtering transparency note in Methodology section
- `src/app/rankings/methodology/page.tsx` — Data Quality section + v1.2 changelog
- `src/lib/ai/analysis.ts` — Pre-filtered data note in prompt + methodology text

---

## Story 5: Before/After Validation + Score Logging

**As a** ChainSights operator
**I need** to see the impact of wallet filtering on GVS scores
**So that** I can validate the improvement and communicate changes

### Acceptance Criteria

- [x] AC-5.1: One-time validation script: run GVS calculation for Lido **with and without** filtering, log both scores
- [x] AC-5.2: Log output shows per-component impact: HPR before/after, PDI before/after, DEI before/after, GPI before/after, GVS before/after
- [x] AC-5.3: Log shows which wallets were excluded with their labels (for manual verification)
- [x] AC-5.4: Run validation for 20 DAOs (Lido + 19 others) to confirm cross-DAO benefit
- [x] AC-5.5: Results documented — writes to `data/wallet-filtering-validation.json`

### File List

- `scripts/validate-wallet-filtering.ts` — Before/after validation script

---

## Story Dependencies

```
Story 1 (DB + API Client)
    ↓
Story 2 (Mapping + Overrides)
    ↓
Story 3 (Pipeline Integration)  ← Core value delivery
    ↓
Story 4 (Transparency)  +  Story 5 (Validation)  ← parallel
```

## Definition of Done

- [x] Lido GVS recalculated with filtering — treasury wallets excluded
- [x] Score change is measurable and explainable
- [x] Methodology page updated
- [x] No regression in existing GVS calculations for DAOs without labeled wallets

---

## Code Review (2026-02-12)

**Reviewer:** Senior Dev AI (adversarial)
**Findings:** 4 HIGH, 4 MEDIUM, 2 LOW → all HIGH/MEDIUM fixed

### Fixes Applied

| ID | Severity | Issue | Fix |
|----|----------|-------|-----|
| H1 | HIGH | PDF report missing wallet filtering transparency note | Added to `report-template.tsx` Methodology section |
| H2 | HIGH | All ACs unchecked despite implementation | Updated epic story file |
| H3 | HIGH | Rate limiting 4x over AC spec (~20 req/s vs 5 req/s) | Reduced batch to 5, delay to 1s |
| H4 | HIGH | PROTOCOL keywords too broad (false excludes on delegates) | Narrowed to contract-specific labels + human activity keyword override |
| M1 | MEDIUM | 'safe' keyword too broad (false FLAG risk) | Changed to 'gnosis safe', 'safe: ', 'safe wallet' |
| M2 | MEDIUM | Sequential DB inserts (N round-trips) | Batch insert with `onConflictDoUpdate` |
| M3 | MEDIUM | O(n) dedup with `.includes()` | Switched to `Set<string>` |
| M4 | MEDIUM | Function name mismatch vs AC-3.1 spec | Documented in story (functional, not renamed) |
| L1 | LOW | Unnecessary `Array.from()` on Map iterator | Removed |
| L2 | LOW | No unit tests | Noted — not addressed in this review |
