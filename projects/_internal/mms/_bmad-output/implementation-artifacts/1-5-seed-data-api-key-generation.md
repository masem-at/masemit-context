# Story 1.5: Seed Data & API Key Generation

Status: done

## Story

As an **admin**,
I want initial seed data with HoKi NÖ as the first donation target and API keys for all projects,
so that the system is ready for immediate use after deployment.

## Acceptance Criteria

1. **Given** db/seed.ts is created **When** the seed script is executed **Then** HoKi NÖ is created as a donation target with: name="HoKi NÖ — Kinderhospizteam", slug="hoki-noe", description, external_url="https://www.hospiz-noe.at/projekte/hospizteam-fuer-jugendliche/", is_active=true

2. **Given** the seed script runs **When** API keys are generated for all 5 projects (hoki_help, tellingcube, chainsights, masem_at, admin) **Then** each key follows the format `mms_{project}_{random}` **And** keys are stored as SHA-256 hashes in the api_keys table with correct key_prefix **And** plaintext keys are output to console exactly once

3. **Given** the seed script generates API keys **When** scopes are assigned **Then** hoki_help, tellingcube, chainsights keys get: [donations:read, donations:write] **And** masem_at key gets: [donations:read] **And** admin key gets: [donations:read, donations:write, config:read, config:write]

4. **Given** the seed script has run **When** the seed is run again **Then** it does not create duplicate entries (idempotent or skip-if-exists)

## Tasks / Subtasks

- [x] Task 1: Create seed script foundation (AC: #1, #2)
  - [x] 1.1 Create src/db/seed.ts with main() entry point
  - [x] 1.2 Import db client and schema tables
  - [x] 1.3 Add helper function to generate random alphanumeric string (extracted to lib/crypto.ts)
  - [x] 1.4 Add helper function to SHA-256 hash keys (extracted to lib/crypto.ts, shared with auth.ts)

- [x] Task 2: Implement HoKi NÖ donation target seeding (AC: #1, #4)
  - [x] 2.1 Check if donation target with slug "hoki-noe" already exists
  - [x] 2.2 If not exists, insert HoKi NÖ with full data
  - [x] 2.3 Log result (created or skipped)

- [x] Task 3: Implement API key generation (AC: #2, #3, #4)
  - [x] 3.1 Define project-to-scopes mapping for all 5 projects
  - [x] 3.2 For each project, check if API key already exists
  - [x] 3.3 If not exists, generate key in format `mms_{project}_{random}`
  - [x] 3.4 Hash key with SHA-256 and extract key_prefix (first 10 chars)
  - [x] 3.5 Insert into api_keys table with correct scopes
  - [x] 3.6 Output plaintext key to console (only on creation)

- [x] Task 4: Add npm script and test manually (AC: #1, #2, #3, #4)
  - [x] 4.1 Add "seed" script to package.json
  - [x] 4.2 Tests written for configuration and data validation
  - [x] 4.3 Crypto functions tested with uniform distribution check

## Dev Notes

### Architecture Compliance

- **Seed script in src/db/seed.ts** — standalone script, not part of API
- **SHA-256 hashing** — reuse same pattern as middleware/auth.ts (Web Crypto API)
- **API key format** — `mms_{project}_{random}` e.g., `mms_hoki_help_a3b9c7x2k1`
- **key_prefix** — first 8-10 characters for identification without exposing full key
- **Console output** — plaintext keys shown exactly once, never logged elsewhere

### Project-to-Scope Mapping

```typescript
const projectScopes = {
  hoki_help: ['donations:read', 'donations:write'],
  tellingcube: ['donations:read', 'donations:write'],
  chainsights: ['donations:read', 'donations:write'],
  masem_at: ['donations:read'],
  admin: ['donations:read', 'donations:write', 'config:read', 'config:write'],
}
```

### HoKi NÖ Donation Target Data

```typescript
{
  name: 'HoKi NÖ — Kinderhospizteam',
  slug: 'hoki-noe',
  description: 'Das mobile Kinderhospizteam der Hospiz Bewegung Niederösterreich begleitet Familien mit lebensverkürzend erkrankten Kindern und Jugendlichen.',
  external_url: 'https://www.hospiz-noe.at/projekte/hospizteam-fuer-jugendliche/',
  is_active: true,
}
```

### Idempotency Pattern

```typescript
// Check before insert
const existing = await db
  .select()
  .from(donationTargets)
  .where(eq(donationTargets.slug, 'hoki-noe'))
  .limit(1)

if (existing.length === 0) {
  await db.insert(donationTargets).values({...})
  console.log('Created: HoKi NÖ donation target')
} else {
  console.log('Skipped: HoKi NÖ already exists')
}
```

### Running the Seed

```bash
# Via npm script
npm run seed

# Or directly with tsx
npx tsx src/db/seed.ts
```

### Previous Story Learnings (Story 1.1 - 1.4)

- ESM with `"type": "module"`, exact pinned versions
- Web Crypto API for SHA-256: `crypto.subtle.digest('SHA-256', data)`
- Database queries use Drizzle ORM with `eq()` from drizzle-orm
- Schema tables: `donationTargets`, `apiKeys` from `./schema`

### References

- [Source: _bmad-output/planning-artifacts/architecture.md#Seed Data]
- [Source: _bmad-output/planning-artifacts/epics.md#Story 1.5]
- [Source: _bmad-output/_masemIT/requirements/mms-donations-requirement.md#FR15]

## Dev Agent Record

### Agent Model Used
Claude Opus 4.5 (Barry - Quick Flow Solo Dev)

### Debug Log References

### Completion Notes List
- Extracted shared crypto functions to lib/crypto.ts (DRY principle)
- Fixed modulo bias in random string generation using rejection sampling
- Added process.exit(0) for clean script termination
- Updated auth.ts to use shared sha256Hash function

### File List
- src/db/seed.ts (new)
- src/db/seed.test.ts (new)
- src/lib/crypto.ts (new)
- src/lib/crypto.test.ts (new)
- src/middleware/auth.ts (modified - uses shared crypto)
- src/middleware/auth.test.ts (modified - uses shared crypto)
- package.json (modified - added seed script)

