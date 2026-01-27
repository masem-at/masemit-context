# Story 2.4: Add Filter and Sort Functionality

**Status:** ready-for-dev
**Epic:** 2 - Public Rankings Leaderboard (MVP)
**Story ID:** 2.4
**Created:** 2025-12-22
**Ultimate Context Engine Analysis:** ✅ Complete

---

## 📋 Story

**As a** user,
**I want** to filter DAOs by category and sort by different columns,
**So that** I can find specific types of DAOs or view rankings by different criteria.

---

## ✅ Acceptance Criteria

### Given-When-Then Format (from Epic 2)

**Given** the leaderboard is displayed
**When** I interact with filter and sort controls
**Then** category filter dropdown includes options:
  - All (default - shows all DAOs)
  - DeFi
  - Infrastructure
  - Public Goods
**And** clicking a category filters the table to show only DAOs in that category (FR18)
**And** sort controls allow sorting by:
  - Rank (default, ascending by GVS score descending)
  - Score (GVS score descending/ascending toggle)
  - Change (absolute value of change, descending)
  - Category (alphabetical A-Z/Z-A toggle)
**And** clicking a sort column toggles ascending/descending order
**And** `aria-sort` attribute updates to reflect current sort state (NFR-ACCESS-10)
**And** screen reader announces "Currently sorted by score, descending" on sort action (NFR-ACCESS-11)
**And** filter/sort state persists in URL query params for shareability (FR19)
**And** URL params update without page reload (client-side navigation)
**And** shared links with query params load with correct filter/sort applied
**And** controls are client components with 'use client' directive
**And** server-side data filtering/sorting for SEO and performance

**Implementation Notes (from Epic):**
- Component: `src/components/LeaderboardFilters.tsx` (Client Component)
- Use Next.js `useSearchParams` and `useRouter` for URL state
- Server Component reads query params and filters/sorts server-side for SEO

---

## 🎯 Critical Developer Context

### 🚨 ARCHITECTURE REALITY CHECK: Server vs Client Components

**CRITICAL: RankingsTable is Currently a SERVER COMPONENT**

**What Story 2.1-2.3 Built:**
- **RankingsTable** (`src/components/RankingsTable.tsx`): Server Component (NO 'use client')
- **Rankings Page** (`src/app/rankings/page.tsx`): Server Component (async function)
- **Data Flow**: Page fetches → passes props → RankingsTable renders
- **NO client-side interactivity** - purely static display

**What Story 2.4 REQUIRES:**
- **Client Component for filter/sort controls** - NEW component needed
- **URL state management** - useSearchParams + useRouter hooks
- **Server-side filtering/sorting** - Modify `getLeaderboardRankings()` to accept params
- **Hybrid approach**: Client controls + Server data processing

**Do NOT convert RankingsTable to Client Component!**
- Keep RankingsTable as Server Component
- Create NEW `LeaderboardFilters.tsx` Client Component for controls
- Pass filtered/sorted data to RankingsTable via props

---

### 📚 Previous Story Intelligence (Stories 2.1-2.3)

**Story 2.1 - Leaderboard Server Component** (done):
- Created `/rankings` page as async Server Component
- Established data fetching pattern: `getLeaderboardRankings()`
- ISR caching: `revalidate = 3600` (1 hour)
- No query param handling yet

**Story 2.2 - RankingsTable UI Component** (done):
- Built `RankingsTable.tsx` as Server Component (262 lines)
- Exports: `SCORE_THRESHOLDS`, `getScoreLabel()`, `RankingsTable` component
- Accepts `rankings: LeaderboardRanking[]` prop
- Semantic HTML table with responsive design
- 38 unit tests for component rendering + accessibility

**Story 2.3 - Accessibility Patterns** (done):
- Added CSS gradient patterns for color-blind users
- Enhanced `getScoreLabel()` with bgPattern property
- 47 tests (all passing)
- **KEY LEARNING**: Keep Server Components when possible - only add 'use client' when needed

---

### 🏗️ Technical Architecture Analysis

#### Current Data Flow (Stories 2.1-2.3)

```
/rankings page (Server Component)
    ↓ async fetch
getLeaderboardRankings() - Returns all DAOs sorted by GVS desc
    ↓ props
<RankingsTable rankings={rankings} /> - Static display
```

#### New Data Flow (Story 2.4)

```
/rankings page (Server Component)
    ↓ await searchParams promise
Extract: category, sortBy, sortOrder from URL query params
    ↓ pass params
getLeaderboardRankings(category, sortBy, sortOrder) - Server-side filter/sort
    ↓ props
<div>
  <LeaderboardFilters /> - Client Component for controls
  <RankingsTable rankings={filteredRankings} /> - Server Component display
</div>
```

#### Established Patterns from Admin Page

**Reference Implementation:** `src/components/admin/search-sort-bar.tsx` (Client Component)

**Pattern 1: Client Component with URL State**
```typescript
'use client'
import { useRouter, useSearchParams } from 'next/navigation'
import { useTransition } from 'react'

const router = useRouter()
const searchParams = useSearchParams()
const [isPending, startTransition] = useTransition()

const createQueryString = useCallback(
  (params: Record<string, string>) => {
    const newParams = new URLSearchParams(searchParams.toString())
    for (const [key, value] of Object.entries(params)) {
      if (value) {
        newParams.set(key, value)
      } else {
        newParams.delete(key)
      }
    }
    return newParams.toString()
  },
  [searchParams]
)

const handleFilterChange = (category: string) => {
  startTransition(() => {
    router.push(`/rankings?${createQueryString({ category })}`)
  })
}
```

**Pattern 2: Server Component with Promise-based searchParams**
```typescript
// src/app/rankings/page.tsx
interface PageProps {
  searchParams: Promise<{ category?: string; sortBy?: string; sortOrder?: 'asc' | 'desc' }>
}

export default async function RankingsPage({ searchParams }: PageProps) {
  const params = await searchParams
  const category = params.category || 'all'
  const sortBy = params.sortBy || 'rank'
  const sortOrder = params.sortOrder || 'asc'

  const rankings = await getLeaderboardRankings({
    category,
    sortBy,
    sortOrder
  })

  return (
    <>
      <LeaderboardFilters />
      <RankingsTable rankings={rankings} />
    </>
  )
}
```

**Pattern 3: Server-side Filtering**
```typescript
// src/lib/db/queries/leaderboard.ts
export async function getLeaderboardRankings(options?: {
  category?: string
  sortBy?: string
  sortOrder?: 'asc' | 'desc'
}): Promise<LeaderboardRanking[]> {
  const { category = 'all', sortBy = 'rank', sortOrder = 'asc' } = options || {}

  // 1. Fetch DAOs (filter by category if specified)
  let query = db.select().from(daos).where(eq(daos.isOptedOut, false))
  if (category !== 'all') {
    query = query.where(eq(daos.category, category))
  }

  // 2. Get latest snapshots
  // 3. Build rankings array
  // 4. Sort based on sortBy and sortOrder

  return rankings
}
```

---

### 📦 Data Structure Reference

#### LeaderboardRanking Interface (from Story 2.1)

**Source:** `src/lib/db/queries/leaderboard.ts` (lines 13-28)

```typescript
export interface LeaderboardRanking {
  id: string                          // UUID
  name: string                        // DAO display name
  slug: string                        // URL-friendly identifier
  category: 'defi' | 'infrastructure' | 'public_goods'  // For filtering
  snapshotSpace: string               // Snapshot.org space ID
  gvsScore: string                    // DECIMAL as string, 0-10 scale
  hprScore: string | null             // Component scores (nullable)
  deiScore: string | null
  pdiScore: string | null
  gpiScore: string | null
  confidenceLevel: 'high' | 'medium' | 'low'
  dataCompleteness: string            // DECIMAL as string, 0-100 scale
  snapshotDate: Date                  // Latest calculation timestamp
}
```

#### Category Enum Values

**Source:** `src/lib/db/schema.ts` (DAO category enum)

```typescript
export const daoCategory = pgEnum('dao_category', [
  'defi',
  'infrastructure',
  'public_goods'
])
```

**Display Labels** (from Story 2.2):
```typescript
const CATEGORY_LABELS: Record<LeaderboardRanking['category'], string> = {
  'defi': 'DeFi',
  'infrastructure': 'Infrastructure',
  'public_goods': 'Public Goods'
}
```

---

### 🔧 Implementation Requirements

#### Component: LeaderboardFilters.tsx (NEW - Client Component)

**Location:** `src/components/LeaderboardFilters.tsx`

**Requirements:**
- `'use client'` directive at top of file
- Import: `useRouter`, `useSearchParams` from `next/navigation`
- Import: `useTransition` from `react`
- Category dropdown with 4 options: All, DeFi, Infrastructure, Public Goods
- Sort controls: Buttons or dropdown for 4 sort options
- Visual loading indicator when `isPending` is true
- Accessible: ARIA labels, keyboard navigation, focus management
- Screen reader announcements for sort changes

**UI Elements:**
1. **Category Filter Dropdown**
   - Label: "Filter by Category"
   - Options: All (default), DeFi, Infrastructure, Public Goods
   - Controlled component synced with URL param `?category=`

2. **Sort Controls**
   - Option 1: Rank (default - GVS score descending)
   - Option 2: Score (GVS score with asc/desc toggle)
   - Option 3: Change (absolute change value descending)
   - Option 4: Category (alphabetical with A-Z/Z-A toggle)
   - Current sort indicated visually (active state)
   - `aria-sort="ascending|descending|none"` on active column

3. **Loading State**
   - Show subtle spinner or "Loading..." when `isPending === true`
   - Disable controls during transition

#### Modified: rankings/page.tsx (Server Component)

**Location:** `src/app/rankings/page.tsx`

**Changes:**
1. Add `searchParams` prop to PageProps interface
2. Await the promise to extract query params
3. Pass params to `getLeaderboardRankings()`
4. Render `<LeaderboardFilters />` above `<RankingsTable />`

**Example:**
```typescript
interface PageProps {
  searchParams: Promise<{
    category?: string
    sortBy?: string
    sortOrder?: 'asc' | 'desc'
  }>
}

export default async function RankingsPage({ searchParams }: PageProps) {
  const params = await searchParams
  const category = params.category || 'all'
  const sortBy = params.sortBy || 'rank'
  const sortOrder = params.sortOrder || 'asc'

  const rankings = await getLeaderboardRankings({
    category,
    sortBy,
    sortOrder
  })

  if (rankings.length === 0) {
    return <EmptyState message="No DAOs found for selected filters" />
  }

  return (
    <div>
      <LeaderboardFilters />
      <RankingsTable rankings={rankings} />
    </div>
  )
}
```

#### Modified: lib/db/queries/leaderboard.ts

**Location:** `src/lib/db/queries/leaderboard.ts`

**Function Signature Change:**
```typescript
export async function getLeaderboardRankings(options?: {
  category?: string
  sortBy?: 'rank' | 'score' | 'change' | 'category'
  sortOrder?: 'asc' | 'desc'
}): Promise<LeaderboardRanking[]>
```

**Implementation Steps:**
1. **Destructure options** with defaults
2. **Filter by category** if not 'all':
   ```typescript
   if (category !== 'all') {
     daoQuery = daoQuery.where(eq(daos.category, category as DaoCategory))
   }
   ```
3. **Sort rankings array** based on sortBy:
   - `rank`: Sort by gvsScore (descending for asc, ascending for desc - inverted)
   - `score`: Sort by gvsScore (descending for desc, ascending for asc)
   - `change`: Sort by absolute value of change (descending for desc)
   - `category`: Sort by category label alphabetically (A-Z for asc, Z-A for desc)

**Existing Logic to Preserve:**
- Opt-out filtering: `where(eq(daos.isOptedOut, false))`
- Confidence filtering: `if (completeness < 50) continue`
- Latest snapshot fetching: `getLatestSnapshots(daoIds)`
- Default sort: GVS descending (when sortBy === 'rank')

---

### 🧪 Testing Requirements

#### Unit Tests for LeaderboardFilters Component

**File:** `tests/unit/components/leaderboard-filters.test.tsx` (NEW)

**Test Coverage:**
1. **Rendering:**
   - Renders category dropdown with 4 options
   - Renders sort controls with 4 options
   - Shows correct initial state from URL params

2. **Category Filtering:**
   - Clicking "DeFi" updates URL to `?category=defi`
   - Clicking "All" removes category param
   - Category persists after page reload

3. **Sort Controls:**
   - Clicking "Score" updates URL to `?sortBy=score&sortOrder=desc`
   - Toggle changes sortOrder between asc/desc
   - Multiple clicks toggle between asc/desc
   - aria-sort attribute updates correctly

4. **URL State:**
   - createQueryString merges params correctly
   - Existing params preserved when adding new param
   - Empty values remove param from URL

5. **Loading State:**
   - isPending shows loading indicator
   - Controls disabled during transition

6. **Accessibility:**
   - Keyboard navigation works (Tab, Enter, Space)
   - Screen reader labels present
   - Focus management correct

**Target:** 20-25 tests minimum

#### Integration Tests for getLeaderboardRankings

**File:** `tests/integration/db/leaderboard-filter-sort.test.ts` (NEW)

**Test Coverage:**
1. **Category Filtering:**
   - Filter by 'defi' returns only DeFi DAOs
   - Filter by 'all' returns all DAOs
   - Filter by 'public_goods' returns only Public Goods DAOs

2. **Sorting:**
   - sortBy='rank' + sortOrder='asc' returns GVS descending
   - sortBy='score' + sortOrder='desc' returns GVS descending
   - sortBy='score' + sortOrder='asc' returns GVS ascending
   - sortBy='category' + sortOrder='asc' returns alphabetical A-Z
   - sortBy='change' + sortOrder='desc' returns largest change first

3. **Combined Filtering + Sorting:**
   - category='defi' + sortBy='score' + sortOrder='asc'
   - Verify results match both filters

**Target:** 10-12 tests minimum

#### E2E Tests (Optional - High Value)

**File:** `tests/e2e/rankings-filter-sort.spec.ts` (NEW)

**Test Flows:**
1. Navigate to /rankings
2. Click "DeFi" filter → verify URL and visible DAOs
3. Click "Score" sort → verify URL and order
4. Copy URL → open in new tab → verify same state
5. Verify keyboard navigation works

**Target:** 3-5 E2E tests

---

### 🚨 Common Mistakes to Avoid

#### ❌ DON'T: Convert RankingsTable to Client Component

```typescript
// ❌ WRONG - Don't add 'use client' to RankingsTable.tsx
'use client'
export default function RankingsTable({ rankings }: RankingsTableProps) {
  // This breaks server-side rendering and ISR caching
}
```

#### ✅ DO: Create Separate Client Component for Controls

```typescript
// ✅ CORRECT - New file: LeaderboardFilters.tsx
'use client'
export default function LeaderboardFilters() {
  const router = useRouter()
  const searchParams = useSearchParams()
  // Interactive controls here
}
```

#### ❌ DON'T: Use client-side filtering

```typescript
// ❌ WRONG - Filtering in client component
'use client'
function LeaderboardFilters({ rankings }: { rankings: LeaderboardRanking[] }) {
  const [filtered, setFiltered] = useState(rankings)
  // Client-side filtering breaks SEO and wastes bandwidth
}
```

#### ✅ DO: Use URL params + server-side filtering

```typescript
// ✅ CORRECT - Server-side filtering
export default async function RankingsPage({ searchParams }: PageProps) {
  const params = await searchParams
  const rankings = await getLeaderboardRankings({
    category: params.category,
    sortBy: params.sortBy
  })
  // SEO-friendly, efficient, shareable
}
```

#### ❌ DON'T: Break existing ISR caching

```typescript
// ❌ WRONG - Don't remove revalidate
// export const revalidate = 3600  // DELETED - breaks caching!
```

#### ✅ DO: Keep ISR caching with dynamic params

```typescript
// ✅ CORRECT - ISR caching still works with query params
export const revalidate = 3600  // Cache for 1 hour
// Next.js caches each unique combination of query params
```

#### ❌ DON'T: Forget to handle empty results

```typescript
// ❌ WRONG - No empty state
return <RankingsTable rankings={[]} />  // Shows empty table
```

#### ✅ DO: Show meaningful empty state

```typescript
// ✅ CORRECT - User-friendly empty state
if (rankings.length === 0) {
  return (
    <div className="text-center py-12">
      <p>No DAOs found matching your filters.</p>
      <button onClick={() => router.push('/rankings')}>Clear Filters</button>
    </div>
  )
}
```

#### ❌ DON'T: Use synchronous searchParams (Next.js 14 breaking change)

```typescript
// ❌ WRONG - Old Next.js 13 pattern
export default async function RankingsPage({ searchParams }: PageProps) {
  const category = searchParams.category  // ERROR: searchParams is Promise
}
```

#### ✅ DO: Await searchParams promise (Next.js 14 pattern)

```typescript
// ✅ CORRECT - Next.js 14 pattern
export default async function RankingsPage({ searchParams }: PageProps) {
  const params = await searchParams  // Await the promise first
  const category = params.category  // Now you can access values
}
```

---

### 📐 Architecture Compliance

#### Server Components Pattern (from project_context.md)

**Rule:** Use Server Components by default, add 'use client' only when needed

**Compliance:**
- ✅ Keep RankingsTable as Server Component
- ✅ Keep rankings page as Server Component
- ✅ NEW LeaderboardFilters as Client Component (needs hooks)
- ✅ Data fetching in Server Component (getLeaderboardRankings)

#### Data Fetching Pattern (from project_context.md)

**Rule:** Use async Server Components for data fetching, not useEffect

**Compliance:**
- ✅ Fetch in page component: `await getLeaderboardRankings()`
- ✅ No client-side data fetching
- ✅ No useEffect for data loading
- ✅ ISR caching preserved: `revalidate = 3600`

#### TypeScript Strict Mode (from project_context.md)

**Rule:** All functions must have explicit return types

**Compliance:**
- ✅ `getLeaderboardRankings(): Promise<LeaderboardRanking[]>`
- ✅ `createQueryString(): string`
- ✅ Event handlers: `(): void`

#### Import Patterns (from project_context.md)

**Rule:** Use @/* alias for all src/ imports

**Compliance:**
- ✅ `import { useRouter } from 'next/navigation'`
- ✅ `import { getLeaderboardRankings } from '@/lib/db/queries/leaderboard'`
- ✅ `import RankingsTable from '@/components/RankingsTable'`

#### Database Rules (from project_context.md)

**Rule:** Use Drizzle query builder, never raw SQL

**Compliance:**
- ✅ `db.select().from(daos).where(eq(daos.category, category))`
- ✅ No raw SQL strings
- ✅ Type-safe queries with Drizzle operators

#### Testing Rules (from project_context.md)

**Rule:** Unit tests mirror src/ structure, 80% coverage target

**Compliance:**
- ✅ `tests/unit/components/leaderboard-filters.test.tsx`
- ✅ `tests/integration/db/leaderboard-filter-sort.test.ts`
- ✅ Mock external dependencies (useRouter, useSearchParams)
- ✅ Test accessibility (ARIA attributes, keyboard nav)

---

### 🔗 Dependencies

#### Internal Dependencies

**Modified:**
- `src/app/rankings/page.tsx` - Add searchParams handling
- `src/lib/db/queries/leaderboard.ts` - Add filter/sort params

**Created:**
- `src/components/LeaderboardFilters.tsx` - NEW Client Component

**Unchanged:**
- `src/components/RankingsTable.tsx` - Keep as Server Component
- `src/lib/db/schema.ts` - DAO schema already has category enum

#### External Dependencies (Already Installed)

- `next` 14.2.0 - useRouter, useSearchParams hooks
- `react` 18.3.0 - useTransition, useCallback hooks
- `drizzle-orm` 0.45.1 - Database query filtering
- `@testing-library/react` - Component testing (added in Story 2.2)
- `vitest` 4.0.16 - Test runner

**No new dependencies required** ✅

---

### 🎯 Implementation Strategy

#### Phase 1: Modify Server-Side Data Fetching

1. Update `getLeaderboardRankings()` signature to accept options
2. Implement category filtering logic
3. Implement sorting logic (4 sort modes)
4. Add unit tests for filtering/sorting
5. Verify existing tests still pass

#### Phase 2: Update Rankings Page

1. Add PageProps interface with searchParams
2. Await searchParams promise
3. Extract category, sortBy, sortOrder with defaults
4. Pass options to getLeaderboardRankings()
5. Add empty state handling
6. Keep ISR caching: `revalidate = 3600`

#### Phase 3: Create LeaderboardFilters Client Component

1. Create new file with 'use client' directive
2. Implement createQueryString helper
3. Build category dropdown UI
4. Build sort controls UI
5. Add loading state indicator
6. Wire up event handlers with useTransition
7. Add ARIA attributes for accessibility
8. Style with Tailwind (match existing design)

#### Phase 4: Integration

1. Import LeaderboardFilters in rankings page
2. Render above RankingsTable
3. Test URL param updates
4. Test server-side re-rendering
5. Verify ISR caching still works
6. Test shareable links

#### Phase 5: Testing

1. Write unit tests for LeaderboardFilters (20-25 tests)
2. Write integration tests for getLeaderboardRankings (10-12 tests)
3. Write E2E tests for complete flow (3-5 tests)
4. Run full test suite
5. Verify build succeeds

#### Phase 6: Accessibility Validation

1. Test keyboard navigation (Tab, Enter, Space)
2. Test screen reader announcements
3. Verify ARIA attributes correct
4. Test focus management
5. Run axe DevTools audit

---

### 📊 Story Metadata

**Story Key:** `2-4-add-filter-and-sort-functionality`
**Story File:** `docs/sprint-artifacts/2-4-add-filter-and-sort-functionality.md`
**Epic:** Epic 2 - Public Rankings Leaderboard (MVP)
**Dependencies:** Story 2.1 (done), Story 2.2 (done)
**Blocks:** Story 2.5 (expandable rows - independent)
**Estimated Effort:** 4-6 hours (Medium complexity)
**Priority:** P0 - Critical (Core user functionality)

---

## 📝 Implementation Tasks

### Phase 1: Server-Side Data Fetching ✅

- [ ] Update getLeaderboardRankings signature in `src/lib/db/queries/leaderboard.ts`
  - [ ] Add options parameter with category, sortBy, sortOrder
  - [ ] Add TypeScript interface for options
  - [ ] Set default values: category='all', sortBy='rank', sortOrder='asc'
- [ ] Implement category filtering
  - [ ] Add where clause for category if not 'all'
  - [ ] Use Drizzle eq() operator for type safety
- [ ] Implement sorting logic
  - [ ] sortBy='rank': GVS descending (default)
  - [ ] sortBy='score': GVS asc/desc by sortOrder
  - [ ] sortBy='change': Absolute change value descending
  - [ ] sortBy='category': Alphabetical by category label
- [ ] Add JSDoc documentation to function
- [ ] Verify existing getLeaderboardRankings() tests still pass

### Phase 2: Update Rankings Page ✅

- [ ] Update `src/app/rankings/page.tsx`
  - [ ] Add PageProps interface with searchParams: Promise<{...}>
  - [ ] Await searchParams in component function
  - [ ] Extract category, sortBy, sortOrder with defaults
  - [ ] Pass options object to getLeaderboardRankings()
  - [ ] Add empty state handling for 0 results
  - [ ] Keep `export const revalidate = 3600`
- [ ] Import LeaderboardFilters component
- [ ] Render LeaderboardFilters above RankingsTable
- [ ] Test: Build succeeds, no TypeScript errors

### Phase 3: Create LeaderboardFilters Component ✅

- [ ] Create `src/components/LeaderboardFilters.tsx`
  - [ ] Add 'use client' directive at top
  - [ ] Import useRouter, useSearchParams from 'next/navigation'
  - [ ] Import useTransition, useCallback from 'react'
- [ ] Implement createQueryString helper
  - [ ] Use useCallback for memoization
  - [ ] Merge new params with existing searchParams
  - [ ] Remove params with empty values
- [ ] Build category filter dropdown
  - [ ] Label: "Filter by Category"
  - [ ] Options: All, DeFi, Infrastructure, Public Goods
  - [ ] Sync with URL param `category`
  - [ ] handleCategoryChange with startTransition
- [ ] Build sort controls
  - [ ] 4 sort buttons: Rank, Score, Change, Category
  - [ ] Visual indicator for active sort
  - [ ] Toggle sortOrder on second click
  - [ ] handleSortChange with startTransition
- [ ] Add loading indicator
  - [ ] Show when isPending === true
  - [ ] Disable controls during transition
- [ ] Add ARIA attributes
  - [ ] aria-label on controls
  - [ ] aria-sort on active sort button
  - [ ] aria-live for screen reader announcements
- [ ] Style with Tailwind
  - [ ] Match existing design system
  - [ ] Responsive layout (mobile/desktop)

### Phase 4: Integration Testing ✅

- [ ] Test URL param updates
  - [ ] Click DeFi → URL shows ?category=defi
  - [ ] Click Score sort → URL shows ?sortBy=score&sortOrder=desc
  - [ ] Multiple clicks toggle sortOrder
- [ ] Test server-side filtering
  - [ ] category=defi shows only DeFi DAOs
  - [ ] category=all shows all DAOs
- [ ] Test server-side sorting
  - [ ] sortBy=score shows correct order
  - [ ] sortBy=category shows alphabetical order
- [ ] Test shareable links
  - [ ] Copy URL with params
  - [ ] Open in new tab/window
  - [ ] Verify filters applied correctly
- [ ] Test ISR caching
  - [ ] Verify revalidate=3600 still works
  - [ ] Each unique param combination cached separately

### Phase 5: Unit Testing ✅

- [ ] Create `tests/unit/components/leaderboard-filters.test.tsx`
  - [ ] Test: Renders category dropdown with 4 options
  - [ ] Test: Renders sort controls with 4 options
  - [ ] Test: Category filter updates URL param
  - [ ] Test: Sort control updates URL params
  - [ ] Test: Toggle sortOrder on second click
  - [ ] Test: createQueryString merges params correctly
  - [ ] Test: Loading indicator shows when isPending
  - [ ] Test: Controls disabled during transition
  - [ ] Test: Keyboard navigation works (Tab, Enter)
  - [ ] Test: ARIA attributes present and correct
  - [ ] **Target: 20-25 tests minimum**
- [ ] Create `tests/integration/db/leaderboard-filter-sort.test.ts`
  - [ ] Test: Filter by category='defi' returns only DeFi
  - [ ] Test: Filter by category='all' returns all
  - [ ] Test: Sort by rank (GVS descending)
  - [ ] Test: Sort by score ascending
  - [ ] Test: Sort by category alphabetical
  - [ ] Test: Combined filter + sort
  - [ ] **Target: 10-12 tests minimum**

### Phase 6: Accessibility Validation ✅

- [ ] Keyboard navigation
  - [ ] Tab through all controls
  - [ ] Enter/Space activate controls
  - [ ] Focus visible on all interactive elements
- [ ] Screen reader testing
  - [ ] Test with NVDA (Windows) or VoiceOver (Mac)
  - [ ] Verify "Currently sorted by..." announcements
  - [ ] Verify filter change announcements
- [ ] ARIA attributes
  - [ ] aria-label on all controls
  - [ ] aria-sort reflects current sort state
  - [ ] aria-live region for announcements
- [ ] Run axe DevTools audit
  - [ ] 0 violations target
  - [ ] Fix any accessibility issues found
- [ ] Document results in story file

### Phase 7: Build & Verification ✅

- [ ] Run `npm run build` and verify no errors
- [ ] Run full test suite: `npm test`
- [ ] Verify all new tests passing
- [ ] Verify no regressions in existing tests
- [ ] Test on actual /rankings page
- [ ] Verify URL params work end-to-end
- [ ] Check performance (no noticeable slowdown)
- [ ] Commit changes with descriptive message

---

## 🎓 Learning from Git History

**Recent commit patterns (Stories 2.1-2.3):**

1. **Server Component First Pattern:**
   - Stories 2.1-2.3 all used Server Components
   - Only add 'use client' when absolutely necessary (hooks, interactivity)
   - This maintains SEO benefits and reduces JavaScript bundle size

2. **ISR Caching Strategy:**
   - `revalidate = 3600` established in Story 2.1
   - Caches each unique query param combination
   - Don't break caching by removing revalidate

3. **Testing Standards:**
   - Story 2.2: 38 tests for component
   - Story 2.3: 47 tests (added 9 pattern tests)
   - Expect 20-30 new tests for Story 2.4

4. **Accessibility Focus:**
   - Story 2.3 focused heavily on WCAG 2.1 compliance
   - ARIA attributes, screen reader support required
   - Color contrast, keyboard navigation tested

5. **Code Review Standards:**
   - Story 2.3 code review found 11 issues (6 HIGH, 3 MEDIUM, 2 LOW)
   - All HIGH issues must be fixed before done
   - Expect similar adversarial review for Story 2.4

---

## 📚 Reference Documentation

### Source Documents

- **Epic 2 Story 2.4:** `docs/epics.md` lines 1412-1444 - Original story requirements
- **Architecture:** `docs/architecture.md` - Server Component patterns, ISR caching
- **Project Context:** `docs/project_context.md` - TypeScript rules, Next.js patterns, testing standards
- **Admin SearchSortBar:** `src/components/admin/search-sort-bar.tsx` - Reference implementation
- **Admin Page:** `src/app/admin/page.tsx` - Server-side searchParams pattern

### Technical References

- Next.js 14 useSearchParams: https://nextjs.org/docs/app/api-reference/functions/use-search-params
- Next.js 14 useRouter: https://nextjs.org/docs/app/api-reference/functions/use-router
- React useTransition: https://react.dev/reference/react/useTransition
- WCAG 2.1 Keyboard Accessible: https://www.w3.org/WAI/WCAG21/Understanding/keyboard
- ARIA sort attribute: https://www.w3.org/TR/wai-aria-1.2/#aria-sort

### Git Intelligence

- Latest commits (Story 2.3 completion):
  - `702271c` - Code review fixes (11 issues resolved)
  - `0cbde7a` - Accessibility patterns implementation
  - `9233fc0` - Story 2.2 RankingsTable component
- Pattern: Server Components by default, comprehensive testing, accessibility focus
- Code review finds 6-11 issues per story average

---

## 🔮 Future Story Context

**Story 2.5** will add expandable rows:
- Clicking a row expands to show component score breakdown
- Uses `<details>` and `<summary>` HTML5 elements for accessibility
- Independent of Story 2.4 (can proceed in parallel if needed)
- Will work with filtered/sorted data from Story 2.4

**Story 2.6** will add week-over-week change indicators:
- Requires historical data from Story 1.8 (done)
- Shows +/- change in GVS score since last week
- Color-coded indicators (green +, red -, gray no change)
- Story 2.4 sorting by "change" will sort these indicators

---

**Ultimate Context Engine Analysis Complete** ✅
**Status:** ready-for-dev
**Next:** Run `/bmad:bmm:workflows:dev-story` to implement
