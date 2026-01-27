# Admin Insights Page — Technical Spec

**Created:** 2024-12-19
**Author:** Winston (Architect)
**Status:** Ready for Implementation

---

## Overview

New admin page at `/admin/insights` providing analytics views separate from the order/report management workflow.

---

## View 1: Reports by Space ID

### Purpose
Show which DAOs have been analyzed, how many times, and score trends.

### Data Source
Existing `reported_daos` table (already implemented).

### UI Layout

```
┌─────────────────────────────────────────────────────────────┐
│ Reports by DAO                                    [Refresh] │
├─────────────────────────────────────────────────────────────┤
│ Space ID              │ Name      │ Reports │ Avg  │ Last  │
├───────────────────────┼───────────┼─────────┼──────┼───────┤
│ arbitrumfoundation.eth│ Arbitrum  │ 5       │ 2.8  │ 3/10  │
│ ens.eth               │ ENS       │ 3       │ 4.2  │ 4/10  │
│ gitcoindao.eth        │ Gitcoin   │ 2       │ 3.0  │ 3/10  │
└─────────────────────────────────────────────────────────────┘
```

### Features
- Sorted by `reports_generated` DESC (most analyzed first)
- Click row → filter main admin to show orders for that space
- Show `last_reported_at` as relative time ("2 days ago")
- Color-code scores: 1-3 red, 4-6 yellow, 7-10 green

### API
Use existing `GET /api/admin/reported-daos` endpoint.

---

## View 2: Quiz Statistics (UI Deferred)

**Note:** UI deferred until sufficient data collected. This spec covers the collection schema only.

---

## Quiz Collection Schema

### New Table: `quiz_responses`

```typescript
// src/lib/db/schema.ts

export const quizResponses = pgTable('quiz_responses', {
  id: uuid('id').defaultRandom().primaryKey(),
  answers: jsonb('answers').notNull(),        // {q1: "a", q2: "b", ...}
  recommendedTier: text('recommended_tier'),  // "standard" | "deep_dive"
  completed: boolean('completed').default(true).notNull(),
  referrer: text('referrer'),                 // document.referrer or utm_source
  userAgent: text('user_agent'),              // browser/device info (optional)
  createdAt: timestamp('created_at').defaultNow().notNull(),
})
```

### Privacy Notes
- NO email, NO IP address, NO cookies
- `user_agent` is optional, can be omitted if deemed too identifying
- Data is fully anonymous and non-linkable

### Integration Point

**File:** `src/app/quiz/page.tsx` (or quiz completion handler)

After quiz completes:
```typescript
await fetch('/api/quiz-response', {
  method: 'POST',
  body: JSON.stringify({
    answers: quizState.answers,
    recommendedTier: result.tier,
    referrer: document.referrer || null,
  }),
})
```

### New API Endpoint

**File:** `src/app/api/quiz-response/route.ts`

```typescript
export async function POST(request: Request) {
  const { answers, recommendedTier, referrer } = await request.json()

  // No auth required - this is anonymous public data
  await db.insert(quizResponses).values({
    answers,
    recommendedTier,
    referrer,
  })

  return NextResponse.json({ success: true })
}
```

---

## Page Structure

**File:** `src/app/admin/insights/page.tsx`

```
/admin/insights
├── Header: "Insights"
├── Card: Reports by DAO (View 1)
│   └── Table with reported_daos data
├── Card: Quiz Statistics (View 2) [COMING SOON placeholder]
│   └── "Collecting data... X responses so far"
└── Back link to /admin
```

---

## Implementation Order

1. Add `quizResponses` schema + run `db:push`
2. Create `POST /api/quiz-response` endpoint
3. Integrate collection in quiz page
4. Create `/admin/insights` page with View 1
5. Add "Coming Soon" placeholder for View 2
6. (Later) Build View 2 UI when data volume warrants

---

## Acceptance Criteria

### Quiz Collection
- [ ] `quiz_responses` table created
- [ ] API endpoint accepts POST without auth
- [ ] Quiz page sends response on completion
- [ ] No PII stored

### Admin Insights Page
- [ ] Page accessible at `/admin/insights`
- [ ] Protected by admin auth
- [ ] Shows reported DAOs table
- [ ] Sorted by report count
- [ ] Scores color-coded
- [ ] Link back to main admin

---

## Files to Create/Modify

| File | Action |
|------|--------|
| `src/lib/db/schema.ts` | Add `quizResponses` table |
| `src/app/api/quiz-response/route.ts` | NEW - POST endpoint |
| `src/app/quiz/page.tsx` | Add collection call on complete |
| `src/app/admin/insights/page.tsx` | NEW - insights dashboard |

---

## Estimate

| Task | Time |
|------|------|
| Schema + migration | 10 min |
| Quiz collection API | 15 min |
| Quiz integration | 10 min |
| Insights page (View 1) | 45 min |
| Placeholder for View 2 | 5 min |
| **Total** | ~1.5 hours |

---

*Spec complete. Ready for Amelia.*
