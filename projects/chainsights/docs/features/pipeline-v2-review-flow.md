# Feature Spec: Pipeline v2 — Review Flow & Data Export

**Date:** 2025-12-17
**Author:** Mario (via Mary, Business Analyst)
**Status:** Draft — Ready for implementation planning
**Priority:** High

---

## 1. Executive Summary

Enhance the report generation pipeline to generate the PDF **before** review, enabling quality checks on the final deliverable. Add a decline/regenerate loop with structured feedback, and provide customers with both PDF and JSON exports via secure email links.

---

## 2. Current vs. Proposed Flow

### Current Flow
```
Intake → Fetch → Analyze → [DRAFT REVIEW] → Generate PDF → Send
                                ↑
                    (reviewing raw analysis data)
```

### Proposed Flow (v2)
```
Intake → Fetch → Analyze → Generate PDF → [REVIEW] → Deliver
                                             ↓
                                    ┌────────┴────────┐
                                    ↓                 ↓
                                [APPROVE]         [DECLINE]
                                    ↓                 ↓
                              Send Email        Enter Feedback
                           (PDF + JSON links)        ↓
                                            Re-run AI Analysis
                                          (with feedback context)
                                                    ↓
                                            Re-generate PDF
                                                    ↓
                                               [REVIEW]
                                              (loop back)
```

---

## 3. Requirements

| #  | Requirement | Priority |
|----|-------------|----------|
| 1  | Auto-generate PDF after AI analysis completes | Must |
| 2  | Review screen shows PDF preview + JSON download button (admin only) | Must |
| 3  | Approve → Send email with PDF download link + JSON download link | Must |
| 4  | Both PDF and JSON accessible via secure links in delivery email (no login required, time-limited or token-protected) | Must |
| 5  | Decline → Enter revision comments (free text + quick-select templates) | Must |
| 6  | Revision comments appended as `[REVIEWER FEEDBACK]`, AI must address feedback while maintaining factual accuracy from source data | Must |
| 7  | Re-generate PDF → Back to review | Must |
| 8  | Revision history visible in admin UI (version count, previous feedback) | Should |
| 9  | Max 3 auto-iterations, then manual override required to continue | Should |
| 10 | JSON export includes metadata: version, timestamp, DAO name, proposal IDs | Must |
| 11 | Quick feedback templates predefined | Should |

---

## 4. Quick Feedback Templates

Predefined options for common revision scenarios:

| Template | Use Case |
|----------|----------|
| "Score too high" | Decentralization score doesn't match data |
| "Score too low" | Underestimating governance health |
| "Missing whale analysis" | Major holder not identified |
| "Recommendation unclear" | Action items need specificity |
| "Data inconsistency" | Numbers don't add up |

Admin can select multiple templates + add free text for specifics.

---

## 5. Technical Design

### 5.1 Database Changes

**reports table additions:**

| Column | Type | Description |
|--------|------|-------------|
| `version` | integer | Report version (starts at 1, increments on regeneration) |
| `revision_history` | jsonb | Array of `{ version, feedback, timestamp }` |
| `pdf_token` | text | Secure token for PDF download link |
| `json_token` | text | Secure token for JSON download link |
| `token_expires_at` | timestamp | Expiration for download tokens (e.g., 30 days) |

### 5.2 New API Endpoints

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/api/admin/reports/[id]/decline` | POST | Submit decline feedback, trigger re-analysis |
| `/api/download/[token]` | GET | Public download endpoint (PDF or JSON by token) |

### 5.3 Pipeline Status Flow

```
pending → fetching → analyzing → generating → review_pending → approved → sent
                                     ↑              ↓
                                     └── declined ──┘
```

New statuses:
- `generating` — PDF being created
- `review_pending` — Awaiting Mario's review
- `declined` — Feedback submitted, re-analyzing

### 5.4 AI Prompt Modification

When revision feedback exists, append to analysis prompt:

```
[REVIEWER FEEDBACK - MUST ADDRESS]
The previous analysis was declined with the following feedback:
- {selected templates}
- {free text comments}

Please regenerate the analysis addressing this feedback while maintaining
factual accuracy based on the source governance data. Do not fabricate
data to match expectations.
```

### 5.5 Secure Download Links

Format: `https://chainsights.one/api/download/{token}`

- Token: UUID or signed JWT
- Expires: 30 days after generation (configurable)
- No authentication required
- Tracks download count (optional analytics)

---

## 6. Admin UI Changes

### Review Screen (`/admin/reports/[id]`)

```
┌─────────────────────────────────────────────────────────────┐
│ AAVE Governance Report (v2)                    [review_pending] │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─────────────────────────────────────┐                   │
│  │                                     │                   │
│  │         PDF PREVIEW                 │                   │
│  │        (embedded viewer)            │                   │
│  │                                     │                   │
│  └─────────────────────────────────────┘                   │
│                                                             │
│  [Download PDF]  [Download JSON]                           │
│                                                             │
├─────────────────────────────────────────────────────────────┤
│ Revision History (if any)                                   │
│ • v1: "Score too high, Missing whale analysis" - 10:30 AM  │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─────────────────────┐  ┌─────────────────────┐         │
│  │   ✓ APPROVE & SEND  │  │   ✗ DECLINE         │         │
│  └─────────────────────┘  └─────────────────────┘         │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Decline Modal

```
┌─────────────────────────────────────────────────────────────┐
│ Decline Report                                              │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│ Quick Feedback (select all that apply):                    │
│                                                             │
│ [ ] Score too high                                         │
│ [ ] Score too low                                          │
│ [ ] Missing whale analysis                                 │
│ [ ] Recommendation unclear                                 │
│ [ ] Data inconsistency                                     │
│                                                             │
│ Additional Comments:                                        │
│ ┌─────────────────────────────────────────────────────────┐│
│ │ The report doesn't mention wallet 0x123... which holds  ││
│ │ 15% of voting power...                                  ││
│ └─────────────────────────────────────────────────────────┘│
│                                                             │
│ Iteration: 2 of 3 remaining                                │
│                                                             │
│        [Cancel]  [Submit & Regenerate]                     │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 7. Email Template Update

**Subject:** Your Governance Truth Report for {DAO_NAME} is Ready

**Body additions:**
```
Your report is ready for download:

📄 Download PDF Report: {secure_pdf_link}
📊 Download Raw Data (JSON): {secure_json_link}

We believe in transparency. The JSON file contains the complete raw governance
data we collected — the same data our AI used to generate your report.
Verify our analysis yourself or use it for your own research.

These links expire in 30 days. Save your files for future reference.
```

---

## 8. JSON Export Schema

```json
{
  "metadata": {
    "version": 1,
    "generated_at": "2025-12-17T10:30:00Z",
    "dao_name": "AAVE",
    "snapshot_space": "aave.eth",
    "proposals_analyzed": 10,
    "proposal_ids": ["QmXxx...", "QmYyy...", ...]
  },
  "governance_data": {
    "space": { ... },
    "power_distribution": {
      "top1_percentage": 12.5,
      "top5_percentage": 45.2,
      "top10_percentage": 67.8,
      "gini_coefficient": 0.72,
      "unique_voters": 1847
    },
    "top_voters": [
      {
        "address": "0x...",
        "voting_power": 125000,
        "direct_tokens": 50000,
        "delegated_power": 75000,
        "is_delegate": true,
        "is_whale": false,
        "participation_rate": 85.5
      },
      ...
    ],
    "proposals": [ ... ],
    "delegate_mappings": [ ... ]
  }
}
```

---

## 9. Out of Scope (Future)

- Customer self-service dashboard
- API access for programmatic data retrieval
- Webhook notifications on report completion
- Bulk report generation
- Report comparison (v1 vs v2)

---

## 10. Acceptance Criteria

- [ ] PDF auto-generates after AI analysis completes
- [ ] Admin can preview PDF before approval
- [ ] Admin can download JSON from review screen
- [ ] Approve sends email with both PDF and JSON secure links
- [ ] Download links work without authentication
- [ ] Download links expire after 30 days
- [ ] Decline flow captures feedback (templates + free text)
- [ ] Re-analysis incorporates feedback in prompt
- [ ] Revision history shows in admin UI
- [ ] Max 3 iterations enforced
- [ ] JSON includes all required metadata

---

## 11. Implementation Notes

**Recommended order:**
1. Database schema changes (version, tokens, history)
2. Secure download endpoint
3. PDF auto-generation after analyze
4. Admin UI: PDF preview + JSON download
5. Email template with secure links
6. Decline modal + feedback capture
7. Re-analysis with feedback injection
8. Revision history UI
9. Iteration limit enforcement

---

*Document created by Mary (Business Analyst) — 2025-12-17*
