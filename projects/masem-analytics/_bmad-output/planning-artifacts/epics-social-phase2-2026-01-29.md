# Social Media Integration - Phase 2 Epics (E1-E5)

**Date:** 2026-01-29
**Source:** Party Mode Discussion (Mary, John, Winston, Paige, Barry, Sally)
**Status:** Approved by Sempre

---

## Epic 1: Social Table UI Polish

**Goal:** Improve the social posts table with better UX and an inline title field.

### Stories

- **E1-S1**: Add `title TEXT` column to `social_posts` table + migration
- **E1-S2**: Inline-editable title field in SocialTabClient (click-to-edit, PATCH on Enter/blur)
- **E1-S3**: Show date+time (`dd.MM.yy HH:mm`) instead of date-only for published_at
- **E1-S4**: Abbreviate column headers: Imp., Reach, Reac., Comm., Shares, Snaps

### Acceptance Criteria

- Title is editable inline without modal/dialog
- Title persists via API call to update `social_posts.title`
- Datetime format shows hours and minutes
- Table fits on standard screens without horizontal scroll

---

## Epic 2: X Manual Import Form

**Goal:** Enable manual entry of X (Twitter) post analytics via a form.

### Context

X has no CSV/XLSX export. User manually reads analytics from X post pages and enters values.

### Schema Update

- Add `detail_expands INT DEFAULT 0` to `social_metrics`

### Field Mapping (X Analytics -> DB)

| X Analytics Field | DB Field | Notes |
|-------------------|----------|-------|
| Impressions | `impressions` | Exists |
| Engagements | Computed (likes+comments+shares+clicks) | No dedicated field needed |
| Detail expands | `detail_expands` | **New field** |
| Profile visits | `profile_views` | Exists |
| Link clicks | `clicks` | Exists |
| Likes | `likes` | Exists |
| Retweets | `shares` | Exists |
| Replies | `comments` | Exists |

### Stories

- **E2-S1**: DB migration - add `detail_expands` to `social_metrics`
- **E2-S2**: API route `POST /api/social/manual` - accepts JSON with URL + metrics, creates post + metrics snapshot
- **E2-S3**: Manual entry form component (URL field required, date picker, metric number inputs)
- **E2-S4**: Integrate form into Social tab (alongside existing XLSX upload)

### Acceptance Criteria

- Form validates URL as required field
- Submitting creates a `social_posts` entry (platform=x) + `social_metrics` snapshot
- If URL already exists, only a new metrics snapshot is appended (append-only pattern)
- Form resets after successful submission, page refreshes to show new data

---

## Epic 3: Social Date Filter

**Goal:** Apply the dashboard's date range filter to the Social tab, filtering by `published_at`.

### Stories

- **E3-S1**: Pass `days` parameter to `getSocialPosts()` and filter `WHERE published_at >= NOW() - INTERVAL '${days} days'`
- **E3-S2**: Pass `days` parameter to `getSocialSummary()` with same filter
- **E3-S3**: Ensure SocialTab receives and forwards `days` prop (already passed but unused)

### Design Decision

- **Filter field:** `published_at` (not `fetched_at` / import date)
- Posts without `published_at` (NULL) are always shown regardless of filter

### Acceptance Criteria

- Selecting 7d/30d/90d/etc. filters social posts by publish date
- Summary cards reflect filtered data only
- Posts with NULL published_at remain visible

---

## Epic 4: Custom Date Range Picker

**Goal:** Replace days-only filter with start date/time + optional end date/time, globally for all tabs.

### Design Decisions

- Quick buttons (7d, 30d, 90d, 365d) **remain** as shortcuts
- Custom range **added** as additional option
- End date/time is **optional** - empty means "until now"
- URL params: `?start=2026-01-01T00:00&end=2026-01-29T23:59` (or `end` omitted)

### Stories

- **E4-S1**: Update `FilterBar` component - add date/time pickers (start required, end optional)
- **E4-S2**: Update URL param handling in all dashboard pages (`page.tsx`, `[project]/page.tsx`)
- **E4-S3**: Update all query functions to accept `startDate`/`endDate` instead of (or alongside) `days`
- **E4-S4**: Ensure quick buttons translate to start/end internally (e.g., 7d = start=now-7d, end=null)

### Acceptance Criteria

- Quick buttons still work as before
- Custom range with start + end filters all tabs correctly
- Omitting end date defaults to current time
- URL is shareable (params encode the selected range)

---

## Epic 5: X Screenshot OCR (Optional / Future)

**Goal:** Extract X post analytics from screenshots using Vision AI.

### Status: Deferred

Winston's recommendation: Only pursue if volume justifies it. Current volume (27 posts) is well served by manual form (E2).

### Concept

- Upload screenshot of X post analytics page
- Vision API (Claude Vision or similar) extracts: URL, impressions, engagements, detail expands, profile visits, link clicks, likes, retweets, replies
- Parsed values populate form for user confirmation before saving

### Open Questions

- Which Vision API? (Cost, accuracy, latency)
- Confirmation step or auto-save?
- Error handling for unreadable screenshots

### Acceptance Criteria (Draft)

- User uploads screenshot, values are pre-filled in form
- User can correct any misread values before saving
- Works for standard X analytics overlay (as shown in reference screenshot)

---

## Implementation Order

```
E1 (UI Polish) → E2 (X Manual Form) → E3 (Social Date Filter) → E4 (Custom Date Range) → E5 (OCR, optional) → E6 (Mobile Cards)
```

E1 and E2 have no dependencies on each other and could run in parallel.
E3 depends on E4's design decisions but can be implemented with `days` first, then migrated to date range.
E5 is independent and optional.
E6 is independent UX improvement applicable globally.
E7 is a quick enhancement to the social posts table.

---

## Epic 7: Ad Spend Tracking

**Goal:** Track ad spend per social post to distinguish organic vs. promoted content.

### Design Decision (Party Mode: Sally, Paige, Barry)

- **Option B selected:** Single `ad_spend DECIMAL(10,2)` field on `social_posts`
- Value > 0 = promoted post, 0/null = organic
- Currency: always EUR (no currency field needed)
- Display: Inline-editable in table, Badge `€{amount}` when > 0

### Stories

- **E7-S1**: Migration - add `ad_spend DECIMAL(10,2) DEFAULT 0` to `social_posts`
- **E7-S2**: Add `ad_spend` to queries (`getSocialPosts` SELECT)
- **E7-S3**: Inline-editable Ad column in SocialTabClient + PATCH API support
- **E7-S4**: Include `ad_spend` field in SocialManualForm

### Acceptance Criteria

- Ad column shows `€{amount}` badge for promoted posts, empty for organic
- Inline click-to-edit with number input, saves via PATCH
- Manual entry form includes Ad Spend field

---

## Epic 6: Responsive Mobile Cards

**Goal:** Replace horizontal tables with vertical card layouts on mobile (< 768px) for all dashboard tables.

### Context

Tables on mobile require horizontal scrolling and truncate text. Card layout shows same data vertically, optimized for touch screens.

### Pattern

```
<div className="md:hidden">
  <CardView ... />
</div>
<div className="hidden md:block">
  <TableView ... />
</div>
```

### Stories

- **E6-S1**: Create reusable `SocialPostCard` component (mobile card for social posts)
- **E6-S2**: Add responsive switch to `SocialTabClient` (cards on mobile, table on desktop)
- **E6-S3**: Apply pattern to other dashboard tables (Top Pages, Top Referrers, CTA Performance, Events)

### Acceptance Criteria

- No horizontal scrolling on mobile for any dashboard table
- Cards show all relevant data in vertical layout
- Breakpoint switch at `md` (768px)
- Desktop table view unchanged
