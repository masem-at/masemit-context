# Product & Architecture Decisions Log

Running log of key decisions made during development. Captures the *why* behind choices.

---

## 2024-12-19: Anonymous Quiz Analytics

**Participants:** Winston, Sally, Maya, Mario

**Context:** Should we store quiz results anonymously for insights?

**Decision:** YES - implement collection now, dashboard later

**Details:**
- Store answers, recommended tier, referrer, timestamp
- No PII (no email, no IP)
- Add subtle transparency text under quiz: "Results help us improve - no personal data stored"
- Dashboard deferred until we have 50-100+ completions (2-3 weeks)

**Rationale:**
- Low effort (30 min implementation)
- Valuable for understanding user intent and conversion gaps
- Future content asset ("We analyzed 500 DAO community members...")

---

## 2024-12-19: Admin Insights Views

**Participants:** Winston, Sally, Maya, Mario

**Context:** Two new admin views requested:
1. Reports generated per Space ID
2. Quiz statistics summary

**Decision:**

| View | Action | Timeline |
|------|--------|----------|
| Reports per Space ID | Build now | Today |
| Quiz Statistics UI | Defer | After 2-3 weeks of data collection |
| Quiz Collection Schema | Build now | Today |

**UX Decision:** Create separate `/admin/insights` page rather than cluttering main admin dashboard. Main admin = orders/reports. Insights = analytics mode.

**Rationale:**
- View 1 data already exists (reported_daos table)
- View 2 needs data before a dashboard makes sense
- Keep admin UX focused by separating concerns

---

## 2024-12-19: Weekly DAO Rankings Feature

**Participants:** Winston, Sally, Maya, Mary, Mario

**Context:** Public ranking page showing top DAOs by decentralization score

**Decision:** Phased approach

| Phase | What | Status |
|-------|------|--------|
| v0 | Manual Twitter threads | Active - first thread posted |
| v1 | Script generates JSON + tweet draft | Done |
| v2 | Public `/rankings` page | After v0/v1 validate demand |

**Key specs:**
- 10 DAOs in rotation (see `config/ranking-daos.json`)
- Weekly cadence
- Append-only history in `data/rankings/`
- Skip failures, continue with rest

---

## 2024-12-19: Reported DAOs Tracking

**Participants:** Winston, Mario

**Context:** Track which DAOs have been analyzed for autocomplete and analytics

**Decision:** Implemented

**Schema:** `reported_daos` table with:
- space_id (PK)
- name
- reports_generated (counter)
- last_score, avg_score, total_score_sum
- last_reported_at

**Integration:** Upsert called after successful AI analysis in pipeline

---

*Add new decisions above this line*
