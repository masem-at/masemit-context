# Epic: Donations Calculation Refactoring

**Epic ID:** DON-CALC-01
**Status:** In Progress
**Priority:** High
**Created:** 2026-02-04
**Updated:** 2026-02-04

---

## Implementation Checklist

### Story 1: DB Schema Changes ✅
- [x] `donation_config` table dropped
- [x] `project_donation_settings` table created (Drizzle + Neon)
- [x] `target_distribution` table created (Drizzle + Neon)
- [x] Seed script updated with initial config data
- [x] Old donation-config route removed from index.ts
- [x] Schema tests updated

### Story 2: API Contract Changes ✅
- [x] Zod schema: `amount_cents` → `product_price_cents`
- [x] Zod schema: `target_id` optional
- [x] Response type updated for array + metadata

### Story 3: Use Case 1 - Direct Target ✅
- [x] Lookup `project_donation_settings`
- [x] Calculate donation amount
- [x] Create single record for target
- [x] Handle `enabled=false`

### Story 4: Use Case 2 - Distributed ✅
- [x] Lookup `target_distribution` when no target_id
- [x] Split calculation with rounding
- [x] Create N records
- [x] Handle `enabled=false`

### Story 5: Validation ✅
- [x] `product_price_cents > 0`
- [x] `source_project` must exist in settings
- [x] `target_id` validation (if provided)
- [x] Error responses

### Story 6: Docs ✅
- [x] Update `/docs/donations` with new API contract
- [x] Document Use Case 1 vs Use Case 2
- [x] Add calculation example
- [x] Update examples section
- [x] Mark admin endpoints as "coming soon"

### Story 7: Admin Endpoints ✅
- [x] `GET /v1/admin/project-settings`
- [x] `PUT /v1/admin/project-settings/:source_project`
- [x] `GET /v1/admin/target-distribution`
- [x] `PUT /v1/admin/target-distribution/:target_id`
- [x] Warning when distribution_pct sum ≠ 100
- [x] Update `/docs/donations` with admin endpoints

### Story 8: Percentage Override (Discount Code Support) ✅
- [x] Add `donation_percentage_override` to Zod schema
- [x] Update handler logic to use override when provided
- [x] Update tests
- [x] Update `/docs/donations` documentation

---

## Description

Refactor the donations system so that projects report product prices instead of donation amounts. MMS calculates donation amounts based on project settings and distributes to targets according to configured percentages.

### Core Change

Projects send `product_price_cents` (sale price), NOT `amount_cents` (donation). MMS handles:
1. Checking if donations are enabled for the project
2. Calculating donation amount based on project percentage
3. Distributing to one or multiple targets

### Use Cases

| | Use Case 1 | Use Case 2 |
|---|---|---|
| **Projects** | hoki.help | tellingCube, chainSights |
| **target_id** | Required | Not provided |
| **Distribution** | 100% to target | Split per `target_distribution` |
| **Records created** | 1 | N |
| **Override support** | ✅ | ✅ |

**Sub-Use-Case: Discount Code Override**
When a discount code is applied (e.g., 10% off), projects can send `donation_percentage_override` to donate a higher percentage, offsetting the discount with increased charitable contribution.

---

## Stories

### Story 1: Database Schema Changes

**ID:** DON-CALC-01-1
**Status:** To Do

**As a** developer
**I want** the new table structure in place
**So that** donation settings and distribution can be configured

**Acceptance Criteria:**
- [ ] `donation_config` table dropped
- [ ] `project_donation_settings` table created
  - source_project (unique), enabled, percentage
- [ ] `target_distribution` table created
  - target_id (unique FK), distribution_pct, is_active
- [ ] Drizzle schema updated
- [ ] Seed script updated with initial data

**Technical Notes:**
```sql
CREATE TABLE project_donation_settings (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  source_project TEXT UNIQUE NOT NULL,
  enabled BOOLEAN NOT NULL DEFAULT true,
  percentage DECIMAL(5,2) NOT NULL,
  created_at TIMESTAMPTZ DEFAULT now(),
  updated_at TIMESTAMPTZ
);

CREATE TABLE target_distribution (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  target_id UUID NOT NULL REFERENCES donation_targets(id),
  distribution_pct DECIMAL(5,2) NOT NULL,
  is_active BOOLEAN NOT NULL DEFAULT true,
  created_at TIMESTAMPTZ DEFAULT now(),
  updated_at TIMESTAMPTZ,
  UNIQUE(target_id)
);
```

---

### Story 2: API Contract Changes

**ID:** DON-CALC-01-2
**Status:** To Do

**As a** consuming project
**I want** to send product_price_cents instead of amount_cents
**So that** MMS handles all donation calculations

**Acceptance Criteria:**
- [ ] `amount_cents` → `product_price_cents` in request
- [ ] `target_id` becomes optional
- [ ] Response returns array for multi-target donations
- [ ] Response includes `calculated_donation_cents` and `source_percentage`

**API Changes:**
```diff
POST /v1/donations

Request:
- "amount_cents": 2500        // Project calculated
+ "product_price_cents": 2500 // Raw value, MMS calculates

- "target_id": "uuid"         // Required
+ "target_id": "uuid"         // Optional (only Use Case 1)

Response (Use Case 2 - multiple records):
{
  "data": [
    { "id": "uuid-1", "target_id": "hoki-uuid", "amount_cents": 75 },
    { "id": "uuid-2", "target_id": "st-anna-uuid", "amount_cents": 225 }
  ],
  "calculated_donation_cents": 300,
  "source_percentage": 3.0
}
```

---

### Story 3: Use Case 1 - Direct Target Donation

**ID:** DON-CALC-01-3
**Status:** To Do

**As a** project like hoki.help
**I want** to donate 100% to a specific target
**So that** direct donations go to the intended recipient

**Acceptance Criteria:**
- [ ] When `target_id` provided → 1 donation record created
- [ ] Amount = `product_price_cents × project.percentage`
- [ ] Full amount goes to specified target
- [ ] If `enabled=false` → return empty array, no records

**Example Flow:**
```
hoki.help sends:
{
  "source_project": "hoki_help",
  "product_price_cents": 2500,
  "target_id": "hoki-noe-uuid",
  "donation_type": "direct"
}

MMS:
1. Lookup project_donation_settings for hoki_help
   → enabled=true, percentage=100%
2. Calculate: 2500 × 100% = 2500 cents
3. Create 1 record: target=hoki-noe, amount=2500

Response:
{
  "data": [{ "id": "...", "target_id": "hoki-noe-uuid", "amount_cents": 2500 }],
  "calculated_donation_cents": 2500,
  "source_percentage": 100.0
}
```

---

### Story 4: Use Case 2 - Distributed Donation

**ID:** DON-CALC-01-4
**Status:** To Do

**As a** project like tellingCube or chainSights
**I want** donations split across multiple targets
**So that** revenue share is distributed according to MMS config

**Acceptance Criteria:**
- [ ] When `target_id` is null → lookup `target_distribution`
- [ ] Create N donation records (one per active target)
- [ ] Each amount = `donation_amount × target.distribution_pct`
- [ ] Rounding: mathematical (0.5 → up)
- [ ] If `enabled=false` → return empty array, no records

**Example Flow:**
```
tellingCube sends:
{
  "source_project": "tellingCube",
  "product_price_cents": 10000,
  "donation_type": "revenue_share"
}

MMS:
1. Lookup project_donation_settings for tellingCube
   → enabled=true, percentage=3%
2. Calculate donation: 10000 × 3% = 300 cents
3. Lookup target_distribution:
   → hoki-noe: 25%
   → st-anna: 75%
4. Create records:
   → hoki-noe: 300 × 25% = 75 cents
   → st-anna: 300 × 75% = 225 cents

Response:
{
  "data": [
    { "id": "...", "target_id": "hoki-uuid", "amount_cents": 75 },
    { "id": "...", "target_id": "st-anna-uuid", "amount_cents": 225 }
  ],
  "calculated_donation_cents": 300,
  "source_percentage": 3.0
}
```

---

### Story 5: Validation & Error Handling

**ID:** DON-CALC-01-5
**Status:** To Do

**As a** developer
**I want** proper validation on the new fields
**So that** bad data is rejected early

**Acceptance Criteria:**
- [ ] `product_price_cents` required, must be > 0
- [ ] `source_project` must exist in `project_donation_settings`
- [ ] Unknown `source_project` → 400 VALIDATION_ERROR
- [ ] Unknown `target_id` → 400 VALIDATION_ERROR
- [ ] `target_id` provided but target inactive → 400 error

**Test Cases:**
| Input | Expected |
|-------|----------|
| `enabled=false` | 200 OK, `data: []` |
| `target_id` + Use Case 1 | 1 record, 100% to target |
| No `target_id` + Use Case 2 | N records per distribution |
| Unknown `source_project` | 400 VALIDATION_ERROR |
| `product_price_cents <= 0` | 400 VALIDATION_ERROR |
| Rounding 7.5 | → 8 (mathematical) |

---

### Story 6: Documentation Update

**ID:** DON-CALC-01-6
**Status:** Backlog

**As a** API consumer
**I want** updated documentation
**So that** I can integrate correctly

**Acceptance Criteria:**
- [ ] `/docs/donations` updated with new request/response
- [ ] Calculation logic documented with examples
- [ ] Both use cases explained
- [ ] Migration notes for existing integrations

---

### Story 7: Admin Endpoints

**ID:** DON-CALC-01-7
**Status:** To Do

**As an** admin
**I want** endpoints to manage settings
**So that** I can configure without direct DB access

**Acceptance Criteria:**
- [ ] `GET /v1/admin/project-settings` — List all project settings
- [ ] `PUT /v1/admin/project-settings/:source_project` — Update project settings
- [ ] `GET /v1/admin/target-distribution` — List all target distributions
- [ ] `PUT /v1/admin/target-distribution/:target_id` — Update target distribution
- [ ] Warning in response when distribution_pct sum ≠ 100
- [ ] Update `/docs/donations` with admin endpoints section

**Scope:** `config:read` for GET, `config:write` for PUT

**Request/Response Examples:**
```typescript
// GET /v1/admin/project-settings
{
  "data": [
    { "source_project": "hoki_help", "enabled": true, "percentage": "100.00" },
    { "source_project": "tellingcube", "enabled": true, "percentage": "3.00" },
    { "source_project": "chainsights", "enabled": true, "percentage": "5.00" }
  ]
}

// PUT /v1/admin/project-settings/tellingcube
Request: { "enabled": true, "percentage": 5.0 }
Response: { "data": { "source_project": "tellingcube", "enabled": true, "percentage": "5.00", ... } }

// GET /v1/admin/target-distribution
{
  "data": [...],
  "meta": {
    "distribution_sum": 100.0,
    "warning": null  // or "Distribution sum is 85%, should be 100%"
  }
}
```

---

### Story 8: Percentage Override (Discount Code Support)

**ID:** DON-CALC-01-8
**Status:** To Do

**As a** project like tellingCube or chainSights
**I want** to override the donation percentage for specific transactions
**So that** discount code purchases can donate a higher percentage

**Context:**
tellingCube and chainSights have a 10% discount code. When applied, the project wants to donate a higher percentage (e.g., 10% instead of 3%) to offset the discount.

**Acceptance Criteria:**
- [ ] New optional field `donation_percentage_override` in request (number, nullable)
- [ ] When `null` or not provided → use project's default percentage from `project_donation_settings`
- [ ] When provided (> 0) → use this percentage for donation calculation
- [ ] No validation that override > default (client decides)
- [ ] Response `source_percentage` shows actually used percentage
- [ ] Works with both Use Case 1 (direct) and Use Case 2 (distributed)
- [ ] Update `/docs/donations` with new field and example

**API Changes:**
```diff
POST /v1/donations

Request:
  "source_project": "tellingcube",
  "product_price_cents": 10000,
+ "donation_percentage_override": 10.0,  // Optional
  "donation_type": "revenue_share",
  ...
```

**Example Flow:**
```
tellingCube sells product for €100 with 10% discount code applied:
{
  "source_project": "tellingcube",
  "product_price_cents": 9000,  // €90 after discount
  "donation_percentage_override": 10.0,
  "donation_type": "revenue_share"
}

MMS:
1. Override provided → use 10% (not project default 3%)
2. Calculate: 9000 × 10% = 900 cents
3. Distribute per target_distribution

Response:
{
  "data": [...],
  "calculated_donation_cents": 900,
  "source_percentage": 10.0  // Shows override was used
}
```

**Test Cases:**
| Input | Expected |
|-------|----------|
| `override: null` | Use project default |
| `override: 10.0` | Use 10%, ignore project default |
| `override: 0` | 400 VALIDATION_ERROR (must be > 0 if provided) |
| `override: -5` | 400 VALIDATION_ERROR |

---

## Technical Decisions

| Topic | Decision | Rationale |
|-------|----------|-----------|
| Rounding | Mathematical (0.5 → up) | `Math.round()` standard behavior |
| DB Schema | `public` | Only 5-10 tables, no namespace needed |
| valid_from/until | Omitted | Not needed for MVP |
| 100% validation | Admin responsibility | No auto-check on insert/update |
| Admin endpoints | Phase 2 | Focus on core functionality first |
| Percentage override | Client-controlled, no validation | Projects decide when to use higher %, MMS doesn't validate override > default |

---

## Database Changes Summary

```diff
Tables:
  donations              ← unchanged
  donation_targets       ← unchanged
  api_keys               ← unchanged
- donation_config        ← DROP
+ project_donation_settings  ← NEW
+ target_distribution        ← NEW
```

---

## Dependencies

- Story 1 (DB Schema) must complete before Stories 2-5
- Story 6 (Docs) can run in parallel after Story 2

---

## Out of Scope

- Admin UI for configuration
- Automatic 100% sum validation
- Time-based percentage changes (valid_from/until)
- Multi-currency distribution calculations
