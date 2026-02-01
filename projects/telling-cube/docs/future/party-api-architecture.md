# masemIT Party API - Architecture Analysis

**Date:** 2025-12-28
**Status:** Draft / Parking Lot
**Author:** Winston (Architect)

---

## Context

Mario wants a shared Party API that all masemIT commercial products can call to store and retrieve "parties" (users, companies, leads, customers).

---

## Scope

### IN SCOPE (Commercial Portfolio)

| Project | Purpose |
|---------|---------|
| **telling-cube** | Customers, leads, invoices |
| **chainsights** | Customers, leads, subscribers |
| **invoice-handler** | Users, vendor contacts |
| **masemit-site** | Contact form leads |
| **tellingDash** | Future product |

### OUT OF SCOPE

| Project | Reason |
|---------|--------|
| **urc-falke** | Club members - separate context |
| **eletat** | Local tool with GDPR tokenization |
| **voting** | Utility script |

---

## Current State Analysis

### Party Data Scattered Across Projects

**Persons (Individuals):**
```
telling-cube:    ContactSubmission.email/firstName/lastName
                 Invoice.customerEmail/customerName

invoice-handler: User.email/name (NextAuth)

chainsights:     orders.customerEmail/customerName
                 leads.email
                 freeQueryLog.email
                 emailSubscribers.email
```

**Organizations (Companies):**
```
telling-cube:    ContactSubmission.company
                 Invoice.companyName/customerVatId
```

### Key Observations

1. **Email is the common identifier** - Every project uses email as primary person identifier
2. **Mixed auth models** - Some anonymous, some OAuth, some local JWT
3. **Duplicated customer data** - Same person could exist in multiple apps
4. **No cross-project linking** - No way to know if a chainsights lead is also a telling-cube customer

---

## Proposed Architecture

### Technology Choice: Supabase

**Why:**
- PostgreSQL (boring, works)
- Built-in Row Level Security
- REST API out of the box
- Free tier generous
- Can add auth later if needed

### Architecture Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                    masemIT Party API                        │
│                  (Supabase Project)                         │
├─────────────────────────────────────────────────────────────┤
│  Tables:                                                    │
│  ├── parties (id, type, email, name, company, metadata)     │
│  ├── party_products (party_id, product, role, status)       │
│  ├── consents (party_id, type, granted_at, withdrawn_at)    │
│  └── audit_log (party_id, action, timestamp)                │
├─────────────────────────────────────────────────────────────┤
│  Access: Row Level Security + API Keys per product          │
└─────────────────────────────────────────────────────────────┘
          │              │              │              │
          ▼              ▼              ▼              ▼
    ┌──────────┐  ┌──────────────┐  ┌──────────┐  ┌─────────┐
    │tellingCube│  │ ChainSights  │  │tellingDash│  │masem.at │
    └──────────┘  └──────────────┘  └──────────┘  └─────────┘
```

---

## Schema Design (Phase 1)

```sql
-- The minimum viable Party
CREATE TABLE parties (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email TEXT UNIQUE NOT NULL,
  first_name TEXT,
  last_name TEXT,
  company TEXT,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- Which products know this party
CREATE TABLE party_products (
  party_id UUID REFERENCES parties(id),
  product TEXT NOT NULL, -- 'tellingcube', 'chainsights', etc.
  role TEXT DEFAULT 'lead', -- 'lead', 'customer', 'subscriber'
  first_seen_at TIMESTAMPTZ DEFAULT NOW(),
  PRIMARY KEY (party_id, product)
);
```

---

## Value Proposition

1. **Cross-Product Intelligence** - Know customer journey across products
2. **Unified Contact Management** - One source of truth
3. **GDPR Compliance** - Centralized consent and deletion
4. **Future-Proofing** - New products instantly have access to customer base
5. **Optional Shared Auth** - Can add later

---

## Next Steps (When Ready)

1. Create new Supabase project for Party API
2. Design and create schema
3. Set up Row Level Security policies
4. Create API keys for each product
5. Build SDK/client library for consuming apps
6. Start with new data (greenfield)
7. Migrate existing data progressively

---

*This document is parked for future reference. Resume when ready to implement.*
