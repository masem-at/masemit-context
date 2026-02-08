# Epic: DON-SUPPORT-01 — Supporters Endpoint & Schema Hardening

**Status:** ✅ COMPLETED
**Created:** 2026-02-04
**Completed:** 2026-02-04
**Owner:** Sempre

---

## Goal

Spender-Wall-Support für Client-Projekte + Auth-Absicherung + Schema-Flexibilität

## Context

hoki.help benötigt eine Spenderliste für die Supporter-Wall. Gleichzeitig werden zwei technische Schulden adressiert:
- Summary-Endpoint ist fälschlicherweise public (Security Fix)
- donor_note als TEXT ist zu restriktiv für flexible Projekt-Daten

## Business Value

- **hoki.help:** Kann Spender-Wall implementieren
- **Alle Projekte:** Flexible Datenstruktur in donor_note (first_name, is_anonymous, custom fields)
- **Security:** Auth-by-Default Policy durchgesetzt

---

## Stories

### Story 2: Schema Migration — donor_note → JSONB

**Priority:** 1 (First)

**Description:**
Migrate `donor_note` column from TEXT to JSONB for flexible project-specific data storage.

**Acceptance Criteria:**
- [x] AC-2.1: Column type changed to JSONB with DEFAULT '{}'
- [x] AC-2.2: Existing TEXT data migrated (null → '{}', non-null → validated or wrapped)
- [x] AC-2.3: Zod schema updated to accept object instead of string
- [x] AC-2.4: Create donation endpoint accepts JSONB donor_note
- [x] AC-2.5: All existing tests pass
- [x] AC-2.6: New test: donation with complex donor_note object

**Technical Notes:**
```sql
ALTER TABLE donations
ALTER COLUMN donor_note TYPE JSONB
USING COALESCE(
  CASE
    WHEN donor_note IS NULL THEN '{}'::jsonb
    WHEN donor_note ~ '^\s*\{.*\}\s*$' THEN donor_note::jsonb
    ELSE jsonb_build_object('note', donor_note)
  END,
  '{}'::jsonb
);

ALTER TABLE donations
ALTER COLUMN donor_note SET DEFAULT '{}';
```

---

### Story 1: Auth-Fix — Summary Endpoint

**Priority:** 2

**Description:**
Secure `GET /v1/donations/summary` endpoint with API key authentication.

**Acceptance Criteria:**
- [x] AC-1.1: Request without API key returns 401
- [x] AC-1.2: Request with valid API key returns 200
- [x] AC-1.3: Existing summary tests updated to include auth
- [x] AC-1.4: New test: 401 without auth

**Technical Notes:**
- Removed `/public/summary` endpoint entirely (was only rate-limited endpoint)
- `/summary` was already auth-protected

---

### Story 3: Supporters Endpoint

**Priority:** 3

**Description:**
New endpoint to retrieve supporter list for wall display, filtered by project.

**Endpoint Spec:**
```
GET /v1/donations/supporters?source_project=hoki_help&limit=50&offset=0

Headers:
  X-API-Key: required

Response 200:
{
  "supporters": [
    {
      "created_at": "2026-02-04T10:30:00Z",
      "donor_note": { "first_name": "Mario", "is_anonymous": false }
    }
  ],
  "pagination": {
    "total": 127,
    "limit": 50,
    "offset": 0
  }
}

Response 401: Missing or invalid API key
Response 400: Missing source_project parameter
```

**Acceptance Criteria:**
- [x] AC-3.1: Endpoint returns supporters filtered by source_project
- [x] AC-3.2: Auth required (401 without API key)
- [x] AC-3.3: source_project query param required (400 if missing)
- [x] AC-3.4: Pagination with limit (default 50, max 100) and offset
- [x] AC-3.5: Response contains only created_at and donor_note (no amounts, no IDs)
- [x] AC-3.6: Sorted by created_at DESC
- [x] AC-3.7: Tests for all scenarios (10 new tests)

**Technical Notes:**
- Add to `services/donations/` module
- Query returns only `created_at`, `donor_note` columns
- No sensitive data exposure (amount_cents, target_id, internal IDs excluded)

---

### Story 4: Documentation Update

**Priority:** 4

**Description:**
Update `/docs/donations` to include Supporters endpoint documentation.

**Acceptance Criteria:**
- [x] AC-4.1: Supporters endpoint documented with request/response examples
- [x] AC-4.2: Pagination parameters documented
- [x] AC-4.3: Auth requirement noted
- [x] AC-4.4: donor_note JSONB structure documented with examples
- [x] AC-4.5: Auth-by-Default policy mentioned in overview

---

### Story 5: Project Bible Update

**Priority:** 5 (Can be done anytime)

**Description:**
Update Project Bible with new decisions and session notes.

**Acceptance Criteria:**
- [x] AC-5.1: Auth-by-Default Policy in Key Decisions Log
- [x] AC-5.2: donor_note JSONB decision in Key Decisions Log
- [x] AC-5.3: Supporters Endpoint decision in Key Decisions Log
- [x] AC-5.4: Session notes for this epic added

---

## Dependencies

```
Story 2 (Schema) → Story 3 (Supporters needs JSONB)
Story 1 (Auth-Fix) → Independent
Story 3 (Supporters) → Story 4 (Docs)
Story 5 (Bible) → Independent
```

## Out of Scope

- Client-side filtering logic (is_anonymous handling in hoki.help)
- Sub-docs per endpoint (keep single /docs/donations file)
- Cursor-based pagination (limit/offset sufficient for now)

---

## Definition of Done

- [x] All stories completed
- [x] All tests passing (203 tests)
- [x] Documentation updated
- [x] Project Bible updated
- [ ] Code reviewed and merged
