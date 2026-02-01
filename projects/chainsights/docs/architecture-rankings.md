# Architecture Document: DAO Rankings Leaderboard

**Created:** December 21, 2025
**Status:** Ready for Implementation
**Architects:** Sally (Solutions Architect) & Winston (Tech Lead)
**PRD Reference:** docs/design/dao-rankings-leaderboard.md

---

## Table of Contents

1. [Executive Summary](#executive-summary)
2. [Technical Overview](#technical-overview)
3. [System Architecture](#system-architecture)
4. [Database Schema](#database-schema)
5. [API Endpoints](#api-endpoints)
6. [Calculation Engine](#calculation-engine)
7. [Automation & Jobs](#automation--jobs)
8. [Frontend Components](#frontend-components)
9. [Integration Points](#integration-points)
10. [Security Considerations](#security-considerations)
11. [Performance & Caching](#performance--caching)
12. [Deployment Strategy](#deployment-strategy)
13. [Testing Strategy](#testing-strategy)
14. [Migration Path](#migration-path)

---

## Executive Summary

### Purpose
Build a public DAO governance leaderboard that displays weekly rankings of DAOs based on governance health scores, enabling viral growth through social proof and competitive positioning while generating leads through "Claim Your Spot" CTAs.

### Key Architectural Decisions

| Decision | Rationale |
|----------|-----------|
| Weekly snapshot model | Governance metrics change slowly; reduces compute costs and API calls |
| Separate `ranking_snapshots` table | Preserves historical data for trend analysis without bloating main tables |
| Opt-in public listing flag | GDPR compliance + controversy mitigation |
| Server-side calculation | Ensures consistent scoring algorithm and prevents gaming |
| Incremental build (Easy → Medium → Full) | Validate demand before building real-time infrastructure |

### Technology Stack
- **Framework:** Next.js 14 (App Router)
- **Database:** PostgreSQL (Neon) with Drizzle ORM
- **Scheduling:** QStash for weekly cron jobs
- **Caching:** Next.js cache + in-memory for 1 hour
- **AI Analysis:** Reuse existing Claude SDK integration

---

## Technical Overview

### Architecture Pattern
The rankings system follows a **periodic snapshot + cache** pattern:

```
┌─────────────────────────────────────────────────────────────┐
│                    Weekly Cron Job                          │
│              (Sunday 00:00 UTC via QStash)                  │
└────────────────────┬────────────────────────────────────────┘
                     │
                     ▼
         ┌───────────────────────┐
         │ Calculation Engine    │
         │ - Query freeQueryLog  │
         │ - Calculate scores    │
         │ - Compute deltas      │
         │ - Store snapshot      │
         └───────────┬───────────┘
                     │
                     ▼
         ┌───────────────────────┐
         │ ranking_snapshots     │
         │ (PostgreSQL)          │
         └───────────┬───────────┘
                     │
                     ▼
         ┌───────────────────────┐
         │ API Endpoints         │
         │ GET /api/rankings     │
         │ (1-hour cache)        │
         └───────────┬───────────┘
                     │
                     ▼
         ┌───────────────────────┐
         │ Frontend Components   │
         │ /rankings page        │
         └───────────────────────┘
```

### Data Flow

1. **Weekly Calculation:**
   - Cron job triggers Sunday midnight UTC
   - Query all `freeQueryLog` entries with `publicListing = true`
   - Calculate weighted scores using methodology (see PRD)
   - Compute rank and delta from previous week
   - Store in `ranking_snapshots` table

2. **Public Display:**
   - `/api/rankings` endpoint fetches latest snapshot
   - Cached for 1 hour to reduce DB load
   - Frontend renders leaderboard with expand/collapse

3. **Opt-In/Opt-Out:**
   - Users opt-in during free tier query via checkbox
   - Users opt-out via `/api/rankings/opt-out` endpoint
   - Next weekly calculation excludes opted-out DAOs

---

## System Architecture

### Component Diagram

```
┌──────────────────────────────────────────────────────────────┐
│                        Frontend Layer                        │
├──────────────────────────────────────────────────────────────┤
│  /rankings                                                   │
│  ├── LeaderboardTable (expandable rows)                     │
│  ├── RankingRow (score, delta, CTA buttons)                 │
│  └── OptOutModal                                             │
│                                                              │
│  /rankings/methodology                                       │
│  └── MethodologyPage (already exists)                       │
└────────────────────────┬─────────────────────────────────────┘
                         │
                         ▼
┌──────────────────────────────────────────────────────────────┐
│                         API Layer                            │
├──────────────────────────────────────────────────────────────┤
│  GET  /api/rankings           (public - fetch leaderboard)  │
│  GET  /api/rankings/history   (public - historical data)    │
│  GET  /api/rankings/methodology (redirect to page)          │
│  POST /api/rankings/opt-out   (public - remove from list)   │
│  POST /api/jobs/generate-rankings (internal - cron)         │
└────────────────────────┬─────────────────────────────────────┘
                         │
                         ▼
┌──────────────────────────────────────────────────────────────┐
│                      Business Logic                          │
├──────────────────────────────────────────────────────────────┤
│  src/lib/rankings/                                           │
│  ├── calculator.ts    (weighted scoring algorithm)          │
│  ├── snapshot.ts      (snapshot creation & retrieval)       │
│  └── types.ts         (TypeScript interfaces)               │
└────────────────────────┬─────────────────────────────────────┘
                         │
                         ▼
┌──────────────────────────────────────────────────────────────┐
│                      Data Layer                              │
├──────────────────────────────────────────────────────────────┤
│  PostgreSQL (Neon)                                           │
│  ├── ranking_snapshots (weekly snapshots)                   │
│  ├── freeQueryLog (source data for rankings)                │
│  └── reportedDaos (DAO metadata)                             │
└──────────────────────────────────────────────────────────────┘
```

---

## Database Schema

### New Table: `ranking_snapshots`

```sql
CREATE TABLE ranking_snapshots (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

  -- Snapshot metadata
  snapshot_week TEXT NOT NULL,              -- 'YYYY-WW' (e.g., '2025-W51')
  snapshot_date DATE NOT NULL,              -- Date of snapshot creation
  created_at TIMESTAMP DEFAULT NOW(),

  -- DAO identification
  dao_id TEXT NOT NULL,                     -- Snapshot space ID (e.g., 'ens.eth')
  dao_name TEXT NOT NULL,                   -- Display name

  -- Ranking data
  rank INTEGER NOT NULL,                    -- Position in leaderboard (1-N)
  rank_previous INTEGER,                    -- Previous week's rank (for delta)

  -- Scores (all 0-10 scale)
  overall_score DECIMAL(3,1) NOT NULL,      -- Weighted final score
  participation_score DECIMAL(3,1),         -- 30% weight
  decentralization_score DECIMAL(3,1),      -- 25% weight
  proposal_quality_score DECIMAL(3,1),      -- 20% weight
  transparency_score DECIMAL(3,1),          -- 15% weight
  execution_score DECIMAL(3,1),             -- 10% weight

  -- Raw metrics (for display in expanded view)
  gini_coefficient REAL,
  nakamoto_coefficient INTEGER,
  voter_turnout_percentage REAL,
  unique_voters INTEGER,
  top5_percentage REAL,
  top10_percentage REAL,

  -- Metadata
  total_daos_ranked INTEGER NOT NULL,       -- How many DAOs in this week's ranking

  -- Indexes
  CONSTRAINT unique_dao_per_week UNIQUE (dao_id, snapshot_week)
);

-- Indexes for performance
CREATE INDEX idx_ranking_snapshots_week ON ranking_snapshots(snapshot_week DESC);
CREATE INDEX idx_ranking_snapshots_dao ON ranking_snapshots(dao_id);
CREATE INDEX idx_ranking_snapshots_rank ON ranking_snapshots(snapshot_week, rank);
```

### Schema Updates to Existing Tables

**`freeQueryLog` table** (already has required fields):
- `publicListing` (boolean) - User opted in to public rankings
- `listedSince` (timestamp) - When they opted in
- `ratingScore` (real) - Overall governance score
- `giniCoefficient` (real)
- `nakamotoCoefficient` (integer)
- `rankingPosition` (integer) - Will be updated weekly

**No changes needed to existing tables.**

### Drizzle ORM Schema Definition

```typescript
// src/lib/db/schema.ts (add to existing file)

export const rankingSnapshots = pgTable('ranking_snapshots', {
  id: uuid('id').defaultRandom().primaryKey(),

  // Snapshot metadata
  snapshotWeek: text('snapshot_week').notNull(),
  snapshotDate: date('snapshot_date').notNull(),
  createdAt: timestamp('created_at').defaultNow().notNull(),

  // DAO identification
  daoId: text('dao_id').notNull(),
  daoName: text('dao_name').notNull(),

  // Ranking data
  rank: integer('rank').notNull(),
  rankPrevious: integer('rank_previous'),

  // Scores
  overallScore: real('overall_score').notNull(),
  participationScore: real('participation_score'),
  decentralizationScore: real('decentralization_score'),
  proposalQualityScore: real('proposal_quality_score'),
  transparencyScore: real('transparency_score'),
  executionScore: real('execution_score'),

  // Raw metrics
  giniCoefficient: real('gini_coefficient'),
  nakamotoCoefficient: integer('nakamoto_coefficient'),
  voterTurnoutPercentage: real('voter_turnout_percentage'),
  uniqueVoters: integer('unique_voters'),
  top5Percentage: real('top5_percentage'),
  top10Percentage: real('top10_percentage'),

  // Metadata
  totalDaosRanked: integer('total_daos_ranked').notNull(),
})

export type RankingSnapshot = typeof rankingSnapshots.$inferSelect
export type NewRankingSnapshot = typeof rankingSnapshots.$inferInsert
```

---

## API Endpoints

### 1. GET `/api/rankings`

**Purpose:** Fetch current week's rankings
**Authentication:** Public (no auth required)
**Cache:** 1 hour in-memory + HTTP cache headers

**Request:**
```typescript
GET /api/rankings?limit=20
```

**Query Parameters:**
| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `limit` | number | 20 | Max number of results |
| `week` | string | current | Specific week (e.g., '2025-W51') |

**Response:**
```typescript
{
  success: true,
  data: {
    snapshotWeek: "2025-W51",
    snapshotDate: "2025-12-21",
    lastUpdated: "2025-12-21T00:05:00Z",
    nextUpdate: "2025-12-28T00:00:00Z",
    totalDaos: 18,
    rankings: [
      {
        rank: 1,
        rankPrevious: 2,
        delta: "+1",              // "+1", "-1", "0", "NEW"
        daoId: "ens.eth",
        daoName: "ENS",
        overallScore: 8.5,
        categoryScores: {
          participation: 9.0,
          decentralization: 8.0,
          proposalQuality: 9.0,
          transparency: 8.0,
          execution: 7.0
        },
        metrics: {
          giniCoefficient: 0.45,
          nakamotoCoefficient: 125,
          top5Percentage: 31,
          uniqueVoters: 1250
        },
        teaser: "Top 5 wallets control 31% of voting power"
      }
    ]
  }
}
```

**Implementation Pattern:**
```typescript
// src/app/api/rankings/route.ts
import { NextResponse } from 'next/server'
import { getRankingsForWeek } from '@/lib/rankings/snapshot'

// Cache for 1 hour
let cachedRankings: any = null
let cacheTimestamp = 0
const CACHE_TTL = 60 * 60 * 1000 // 1 hour

export async function GET(request: Request) {
  const { searchParams } = new URL(request.url)
  const limit = parseInt(searchParams.get('limit') || '20')
  const week = searchParams.get('week') // null = current week

  // Check cache
  if (cachedRankings && Date.now() - cacheTimestamp < CACHE_TTL && !week) {
    return NextResponse.json(cachedRankings, {
      headers: { 'Cache-Control': 'public, max-age=3600' }
    })
  }

  // Fetch from DB
  const rankings = await getRankingsForWeek(week, limit)

  // Update cache
  if (!week) {
    cachedRankings = rankings
    cacheTimestamp = Date.now()
  }

  return NextResponse.json(rankings, {
    headers: { 'Cache-Control': 'public, max-age=3600' }
  })
}
```

---

### 2. GET `/api/rankings/history`

**Purpose:** Fetch historical trend for a specific DAO
**Authentication:** Public
**Cache:** 6 hours

**Request:**
```typescript
GET /api/rankings/history?daoId=ens.eth&weeks=12
```

**Response:**
```typescript
{
  success: true,
  daoId: "ens.eth",
  daoName: "ENS",
  history: [
    {
      week: "2025-W51",
      rank: 1,
      score: 8.5,
      date: "2025-12-21"
    },
    {
      week: "2025-W50",
      rank: 2,
      score: 8.3,
      date: "2025-12-14"
    }
  ]
}
```

---

### 3. POST `/api/rankings/opt-out`

**Purpose:** Remove DAO from public rankings
**Authentication:** Public (validates DAO ID exists)
**Rate Limit:** 10 requests/minute per IP

**Request:**
```typescript
POST /api/rankings/opt-out
{
  daoId: "ens.eth",
  email?: "contact@dao.org" // Optional for confirmation
}
```

**Response:**
```typescript
{
  success: true,
  message: "ens.eth has been removed from public rankings",
  recordsUpdated: 3
}
```

**Implementation:**
- Sets `publicListing = false` in `freeQueryLog` for all records with matching `daoId`
- Next weekly calculation will exclude this DAO
- Invalidates API cache

**Already implemented at:** `src/app/api/rankings/opt-out/route.ts`

---

### 4. POST `/api/jobs/generate-rankings` (Internal)

**Purpose:** Weekly cron job to generate new ranking snapshot
**Authentication:** QStash signature verification
**Trigger:** Weekly (Sunday 00:00 UTC)

**Request:**
```typescript
POST /api/jobs/generate-rankings
Headers:
  Upstash-Signature: <qstash-signature>
```

**Response:**
```typescript
{
  success: true,
  snapshotWeek: "2025-W51",
  daosRanked: 18,
  processingTime: 3421, // ms
  errors: []
}
```

**Implementation:**
1. Verify QStash signature
2. Call `generateWeeklyRankings()` from calculation engine
3. Invalidate API cache
4. Send notification to LogSnag
5. Return summary

---

## Calculation Engine

### Core Algorithm

Located in: `src/lib/rankings/calculator.ts`

### Weighted Scoring Formula

```typescript
overallScore =
  (participation * 0.30) +
  (decentralization * 0.25) +
  (proposalQuality * 0.20) +
  (transparency * 0.15) +
  (execution * 0.10)
```

### Category Scoring Functions

#### 1. Participation Score (30%)

**Metric:** Voter turnout percentage
**Source:** `freeQueryLog.ratingScore` (already calculated)

```typescript
function calculateParticipationScore(voterTurnout: number): number {
  // voterTurnout is 0-100 percentage
  if (voterTurnout >= 40) return 10
  if (voterTurnout >= 30) return 9
  if (voterTurnout >= 20) return 7
  if (voterTurnout >= 10) return 5
  if (voterTurnout >= 5) return 3
  return 1
}
```

#### 2. Decentralization Score (25%)

**Metrics:** Gini coefficient + Nakamoto coefficient
**Source:** `freeQueryLog.giniCoefficient`, `freeQueryLog.nakamotoCoefficient`

```typescript
function calculateDecentralizationScore(
  gini: number,
  nakamoto: number
): number {
  // Gini score (0 = perfect equality, 1 = total inequality)
  let giniScore = 0
  if (gini < 0.5) giniScore = 10
  else if (gini < 0.6) giniScore = 8
  else if (gini < 0.7) giniScore = 6
  else if (gini < 0.85) giniScore = 4
  else giniScore = 2

  // Nakamoto score (min entities for >50% control)
  let nakamotoScore = 0
  if (nakamoto > 100) nakamotoScore = 10
  else if (nakamoto > 50) nakamotoScore = 8
  else if (nakamoto > 20) nakamotoScore = 6
  else if (nakamoto > 10) nakamotoScore = 4
  else nakamotoScore = 2

  // Average the two
  return (giniScore + nakamotoScore) / 2
}
```

#### 3. Proposal Quality Score (20%)

**Approach:** Extract from AI analysis if available, otherwise estimate

```typescript
function calculateProposalQualityScore(
  aiAnalysis: any
): number {
  // If AI analysis exists, extract quality indicators
  if (aiAnalysis?.votingPatterns) {
    const passRate = aiAnalysis.votingPatterns.passRate || 0.85
    const avgDiscussion = aiAnalysis.votingPatterns.avgComments || 10

    // Ideal pass rate: 60-80% (not rubber-stamping)
    let passScore = 0
    if (passRate >= 0.6 && passRate <= 0.8) passScore = 10
    else if (passRate >= 0.5 && passRate <= 0.9) passScore = 7
    else passScore = 4

    // Discussion engagement
    let discussionScore = 0
    if (avgDiscussion > 50) discussionScore = 10
    else if (avgDiscussion > 20) discussionScore = 7
    else if (avgDiscussion > 10) discussionScore = 5
    else discussionScore = 3

    return (passScore + discussionScore) / 2
  }

  // Default: Neutral score if no data
  return 6.0
}
```

#### 4. Transparency Score (15%)

**Metrics:** On-chain voting %, public discussion presence

```typescript
function calculateTransparencyScore(
  onChainPercentage: number,
  hasPublicForum: boolean
): number {
  let score = 0

  // On-chain voting percentage
  if (onChainPercentage === 100) score += 6
  else if (onChainPercentage >= 80) score += 5
  else if (onChainPercentage >= 50) score += 3
  else score += 1

  // Public forum presence
  if (hasPublicForum) score += 4
  else score += 1

  return score
}
```

#### 5. Execution Score (10%)

**Metrics:** Proposal implementation rate

```typescript
function calculateExecutionScore(
  implementationRate: number
): number {
  if (implementationRate >= 90) return 10
  if (implementationRate >= 70) return 8
  if (implementationRate >= 50) return 6
  if (implementationRate >= 30) return 4
  return 2
}
```

### Data Source Strategy

**Phase 1 (MVP):** Use existing data from `freeQueryLog`
- `ratingScore` → Overall score (already calculated by AI)
- `giniCoefficient` → Decentralization
- `nakamotoCoefficient` → Decentralization
- Other metrics: Default values or estimates

**Phase 2 (Enhanced):** Extract from AI analysis JSON
- Query `reports.aiAnalysis` for DAOs with full reports
- Parse detailed metrics (pass rate, discussion counts, etc.)
- Fall back to Phase 1 approach if report unavailable

### Teaser Generation

```typescript
function generateTeaser(metrics: RankingMetrics): string {
  const { top5Percentage, giniCoefficient, nakamotoCoefficient } = metrics

  // Priority: Most interesting metric for this DAO
  if (top5Percentage > 75) {
    return `⚠️ Top 5 wallets control ${top5Percentage.toFixed(0)}% of voting power`
  }

  if (top5Percentage > 50) {
    return `Top 5 wallets control ${top5Percentage.toFixed(0)}% of voting power`
  }

  if (nakamotoCoefficient < 10) {
    return `Only ${nakamotoCoefficient} entities needed for majority control`
  }

  if (giniCoefficient > 0.85) {
    return `High power concentration (Gini: ${giniCoefficient.toFixed(2)})`
  }

  // Positive frame for healthy DAOs
  return `Well-distributed voting power (Top 5: ${top5Percentage.toFixed(0)}%)`
}
```

---

## Automation & Jobs

### Weekly Ranking Generation Job

**File:** `src/app/api/jobs/generate-rankings/route.ts`

**Trigger:** QStash cron schedule
**Schedule:** `0 0 * * 0` (Every Sunday at 00:00 UTC)
**Timeout:** 5 minutes

**Implementation:**

```typescript
import { NextRequest, NextResponse } from 'next/server'
import { verifySignature } from '@upstash/qstash/nextjs'
import { generateWeeklyRankings } from '@/lib/rankings/snapshot'
import { log } from '@/lib/logsnag'

async function handler(req: NextRequest) {
  try {
    console.log('[Rankings Job] Starting weekly ranking generation...')

    // Generate rankings
    const result = await generateWeeklyRankings()

    // Log success to LogSnag
    await log({
      channel: 'rankings',
      event: 'Weekly Rankings Generated',
      description: `Generated ${result.daosRanked} DAO rankings for week ${result.snapshotWeek}`,
      icon: '📊',
      notify: false
    })

    return NextResponse.json({
      success: true,
      ...result
    })
  } catch (error) {
    console.error('[Rankings Job] Error:', error)

    // Log error to LogSnag
    await log({
      channel: 'rankings',
      event: 'Ranking Generation Failed',
      description: error instanceof Error ? error.message : 'Unknown error',
      icon: '❌',
      notify: true
    })

    return NextResponse.json(
      { success: false, error: 'Failed to generate rankings' },
      { status: 500 }
    )
  }
}

export const POST = verifySignature(handler)

export const runtime = 'nodejs'
export const maxDuration = 300 // 5 minutes
```

### Core Ranking Generation Logic

**File:** `src/lib/rankings/snapshot.ts`

```typescript
import { db } from '@/lib/db'
import { rankingSnapshots, freeQueryLog } from '@/lib/db/schema'
import { eq, desc } from 'drizzle-orm'
import { calculateScores, generateTeaser } from './calculator'

export async function generateWeeklyRankings() {
  const startTime = Date.now()

  // 1. Calculate current week identifier
  const now = new Date()
  const weekNumber = getISOWeek(now)
  const year = now.getFullYear()
  const snapshotWeek = `${year}-W${weekNumber.toString().padStart(2, '0')}`

  // 2. Fetch all DAOs opted into public listing
  const eligibleDaos = await db
    .select()
    .from(freeQueryLog)
    .where(eq(freeQueryLog.publicListing, true))

  if (eligibleDaos.length === 0) {
    throw new Error('No DAOs opted into public rankings')
  }

  // 3. Group by DAO (take most recent entry per DAO)
  const daoMap = new Map<string, typeof eligibleDaos[0]>()
  for (const entry of eligibleDaos) {
    const existing = daoMap.get(entry.daoId)
    if (!existing || entry.createdAt > existing.createdAt) {
      daoMap.set(entry.daoId, entry)
    }
  }

  // 4. Calculate scores for each DAO
  const scoredDaos = Array.from(daoMap.values()).map(dao => {
    const scores = calculateScores({
      ratingScore: dao.ratingScore,
      giniCoefficient: dao.giniCoefficient || 0.5,
      nakamotoCoefficient: dao.nakamotoCoefficient || 50,
      // Add more metrics as available
    })

    return {
      daoId: dao.daoId,
      scores,
      metrics: {
        giniCoefficient: dao.giniCoefficient,
        nakamotoCoefficient: dao.nakamotoCoefficient,
        // Extract more metrics from dao entry
      }
    }
  })

  // 5. Sort by overall score descending
  scoredDaos.sort((a, b) => b.scores.overall - a.scores.overall)

  // 6. Fetch previous week's rankings for delta calculation
  const previousWeek = getPreviousWeek(snapshotWeek)
  const previousRankings = await db
    .select()
    .from(rankingSnapshots)
    .where(eq(rankingSnapshots.snapshotWeek, previousWeek))

  const previousRankMap = new Map(
    previousRankings.map(r => [r.daoId, r.rank])
  )

  // 7. Insert new snapshots
  const snapshots = scoredDaos.map((dao, index) => ({
    snapshotWeek,
    snapshotDate: now.toISOString().split('T')[0],
    daoId: dao.daoId,
    daoName: dao.daoId, // TODO: Fetch from reportedDaos table
    rank: index + 1,
    rankPrevious: previousRankMap.get(dao.daoId) || null,
    overallScore: dao.scores.overall,
    participationScore: dao.scores.participation,
    decentralizationScore: dao.scores.decentralization,
    proposalQualityScore: dao.scores.proposalQuality,
    transparencyScore: dao.scores.transparency,
    executionScore: dao.scores.execution,
    giniCoefficient: dao.metrics.giniCoefficient,
    nakamotoCoefficient: dao.metrics.nakamotoCoefficient,
    totalDaosRanked: scoredDaos.length,
  }))

  await db.insert(rankingSnapshots).values(snapshots)

  const processingTime = Date.now() - startTime

  return {
    snapshotWeek,
    daosRanked: snapshots.length,
    processingTime,
    errors: []
  }
}

function getISOWeek(date: Date): number {
  const target = new Date(date.valueOf())
  const dayNr = (date.getDay() + 6) % 7
  target.setDate(target.getDate() - dayNr + 3)
  const firstThursday = target.valueOf()
  target.setMonth(0, 1)
  if (target.getDay() !== 4) {
    target.setMonth(0, 1 + ((4 - target.getDay()) + 7) % 7)
  }
  return 1 + Math.ceil((firstThursday - target.valueOf()) / 604800000)
}

function getPreviousWeek(currentWeek: string): string {
  // Parse "2025-W51" -> "2025-W50"
  const [year, week] = currentWeek.split('-W')
  const weekNum = parseInt(week)
  if (weekNum === 1) {
    return `${parseInt(year) - 1}-W52`
  }
  return `${year}-W${(weekNum - 1).toString().padStart(2, '0')}`
}
```

### QStash Configuration

Add to `.env.local`:
```bash
QSTASH_TOKEN=your_qstash_token
QSTASH_CURRENT_SIGNING_KEY=your_signing_key
QSTASH_NEXT_SIGNING_KEY=your_next_key
```

**Cron setup:** Configure in Vercel or QStash dashboard
```bash
curl -X POST https://qstash.upstash.io/v2/schedules \
  -H "Authorization: Bearer $QSTASH_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "destination": "https://chainsights.one/api/jobs/generate-rankings",
    "cron": "0 0 * * 0"
  }'
```

---

## Frontend Components

### Page Structure

**Route:** `src/app/rankings/page.tsx`

### Component Hierarchy

```
RankingsPage
├── Header (reused)
├── RankingsHero
│   ├── Title + Description
│   ├── Last Updated Badge
│   └── Methodology Link
├── LeaderboardTable
│   ├── RankingRow (repeating)
│   │   ├── RankBadge
│   │   ├── DaoInfo (name + logo)
│   │   ├── ScoreBar
│   │   ├── DeltaIndicator (+/-/→)
│   │   ├── Top5Badge
│   │   └── ExpandButton
│   └── ExpandedDetails (conditional)
│       ├── CategoryScores
│       ├── MetricsGrid
│       └── CTAButtons
│           ├── GetReportButton
│           └── ClaimYourSpotButton
└── Footer (reused)
```

### Key Components

#### 1. LeaderboardTable

```typescript
// src/app/rankings/page.tsx

'use client'

import { useState, useEffect } from 'react'
import { Card } from '@/components/ui/card'
import { Badge } from '@/components/ui/badge'
import { Button } from '@/components/ui/button'
import { ChevronDown, ChevronUp, TrendingUp, TrendingDown, Minus } from 'lucide-react'

interface RankingData {
  rank: number
  rankPrevious: number | null
  daoId: string
  daoName: string
  overallScore: number
  categoryScores: {
    participation: number
    decentralization: number
    proposalQuality: number
    transparency: number
    execution: number
  }
  metrics: {
    giniCoefficient: number
    nakamotoCoefficient: number
    top5Percentage: number
    uniqueVoters: number
  }
  teaser: string
}

export default function RankingsPage() {
  const [rankings, setRankings] = useState<RankingData[]>([])
  const [expandedRows, setExpandedRows] = useState<Set<string>>(new Set())
  const [loading, setLoading] = useState(true)

  useEffect(() => {
    fetch('/api/rankings')
      .then(res => res.json())
      .then(data => {
        setRankings(data.data.rankings)
        setLoading(false)
      })
  }, [])

  const toggleRow = (daoId: string) => {
    const newExpanded = new Set(expandedRows)
    if (newExpanded.has(daoId)) {
      newExpanded.delete(daoId)
    } else {
      newExpanded.add(daoId)
    }
    setExpandedRows(newExpanded)
  }

  if (loading) {
    return <div className="text-center py-20">Loading rankings...</div>
  }

  return (
    <main className="min-h-screen bg-hero-gradient pt-24 pb-16 px-6">
      <div className="max-w-6xl mx-auto">
        {/* Hero Section */}
        <div className="text-center mb-12">
          <h1 className="font-montserrat text-4xl font-bold text-white mb-4">
            DAO Governance Leaderboard
          </h1>
          <p className="text-xl text-gray-300 mb-6">
            Weekly rankings of DAO governance health
          </p>
          <Badge className="bg-aqua/20 text-aqua border-aqua/30">
            Updated: Dec 21, 2025 • Next update: Dec 28
          </Badge>
        </div>

        {/* Leaderboard */}
        <Card className="border-border bg-card overflow-hidden">
          <div className="overflow-x-auto">
            <table className="w-full">
              <thead className="border-b border-border">
                <tr>
                  <th className="px-6 py-4 text-left text-sm font-semibold text-gray-300">
                    Rank
                  </th>
                  <th className="px-6 py-4 text-left text-sm font-semibold text-gray-300">
                    DAO
                  </th>
                  <th className="px-6 py-4 text-left text-sm font-semibold text-gray-300">
                    Score
                  </th>
                  <th className="px-6 py-4 text-left text-sm font-semibold text-gray-300">
                    Change
                  </th>
                  <th className="px-6 py-4 text-left text-sm font-semibold text-gray-300">
                    Top 5 Control
                  </th>
                  <th className="px-6 py-4"></th>
                </tr>
              </thead>
              <tbody>
                {rankings.map((dao) => (
                  <RankingRow
                    key={dao.daoId}
                    dao={dao}
                    isExpanded={expandedRows.has(dao.daoId)}
                    onToggle={() => toggleRow(dao.daoId)}
                  />
                ))}
              </tbody>
            </table>
          </div>
        </Card>

        {/* Methodology Link */}
        <div className="text-center mt-8">
          <a
            href="/rankings/methodology"
            className="text-aqua hover:underline"
          >
            How are these scores calculated?
          </a>
        </div>
      </div>
    </main>
  )
}
```

#### 2. RankingRow Component

```typescript
// src/components/rankings/RankingRow.tsx

function RankingRow({ dao, isExpanded, onToggle }: {
  dao: RankingData
  isExpanded: boolean
  onToggle: () => void
}) {
  const delta = dao.rankPrevious
    ? dao.rankPrevious - dao.rank
    : null

  const getDeltaIcon = () => {
    if (delta === null) return <span className="text-gray-400">NEW</span>
    if (delta > 0) return <TrendingUp className="w-4 h-4 text-green-400" />
    if (delta < 0) return <TrendingDown className="w-4 h-4 text-red-400" />
    return <Minus className="w-4 h-4 text-gray-400" />
  }

  const getDeltaText = () => {
    if (delta === null) return 'New'
    if (delta > 0) return `+${delta}`
    if (delta < 0) return `${delta}`
    return '—'
  }

  const getTop5Color = () => {
    if (dao.metrics.top5Percentage > 75) return 'text-red-400 font-bold'
    if (dao.metrics.top5Percentage > 50) return 'text-yellow-400 font-semibold'
    return 'text-gray-300'
  }

  return (
    <>
      <tr
        className="border-b border-border hover:bg-navy-dark/50 cursor-pointer transition-colors"
        onClick={onToggle}
      >
        <td className="px-6 py-4">
          <div className="flex items-center gap-2">
            <span className="text-xl font-bold text-white">#{dao.rank}</span>
          </div>
        </td>
        <td className="px-6 py-4">
          <div className="font-semibold text-white">{dao.daoName}</div>
          <div className="text-sm text-gray-400">{dao.daoId}</div>
        </td>
        <td className="px-6 py-4">
          <ScoreBar score={dao.overallScore} />
        </td>
        <td className="px-6 py-4">
          <div className="flex items-center gap-1">
            {getDeltaIcon()}
            <span className="text-sm text-gray-300">{getDeltaText()}</span>
          </div>
        </td>
        <td className="px-6 py-4">
          <span className={getTop5Color()}>
            {dao.metrics.top5Percentage.toFixed(0)}%
            {dao.metrics.top5Percentage > 75 && ' ⚠️'}
          </span>
        </td>
        <td className="px-6 py-4">
          <Button variant="ghost" size="sm">
            {isExpanded ? <ChevronUp /> : <ChevronDown />}
          </Button>
        </td>
      </tr>
      {isExpanded && (
        <tr>
          <td colSpan={6} className="px-6 py-6 bg-navy-dark/30">
            <ExpandedDetails dao={dao} />
          </td>
        </tr>
      )}
    </>
  )
}
```

#### 3. ScoreBar Component

```typescript
// src/components/rankings/ScoreBar.tsx

function ScoreBar({ score }: { score: number }) {
  const percentage = (score / 10) * 100
  const color = score >= 7 ? 'bg-green-500' : score >= 4 ? 'bg-yellow-500' : 'bg-red-500'

  return (
    <div className="flex items-center gap-2">
      <div className="w-24 h-2 bg-gray-700 rounded-full overflow-hidden">
        <div
          className={`h-full ${color} transition-all`}
          style={{ width: `${percentage}%` }}
        />
      </div>
      <span className="text-sm font-semibold text-white">
        {score.toFixed(1)}/10
      </span>
    </div>
  )
}
```

#### 4. ExpandedDetails Component

```typescript
// src/components/rankings/ExpandedDetails.tsx

function ExpandedDetails({ dao }: { dao: RankingData }) {
  return (
    <div className="space-y-6">
      {/* Teaser */}
      <div className="text-lg text-gray-300 italic">
        "{dao.teaser}"
      </div>

      {/* Category Scores */}
      <div className="grid grid-cols-2 md:grid-cols-5 gap-4">
        <CategoryScore
          label="Participation"
          score={dao.categoryScores.participation}
          icon="👥"
        />
        <CategoryScore
          label="Decentralization"
          score={dao.categoryScores.decentralization}
          icon="🌐"
        />
        <CategoryScore
          label="Proposal Quality"
          score={dao.categoryScores.proposalQuality}
          icon="📝"
        />
        <CategoryScore
          label="Transparency"
          score={dao.categoryScores.transparency}
          icon="🔍"
        />
        <CategoryScore
          label="Execution"
          score={dao.categoryScores.execution}
          icon="⚡"
        />
      </div>

      {/* Metrics Grid */}
      <div className="grid grid-cols-2 md:grid-cols-4 gap-4 text-sm">
        <Metric label="Gini Coefficient" value={dao.metrics.giniCoefficient.toFixed(2)} />
        <Metric label="Nakamoto Coeff" value={dao.metrics.nakamotoCoefficient} />
        <Metric label="Unique Voters" value={dao.metrics.uniqueVoters.toLocaleString()} />
        <Metric label="Top 5 Control" value={`${dao.metrics.top5Percentage.toFixed(0)}%`} />
      </div>

      {/* CTA Buttons */}
      <div className="flex gap-4">
        <Button
          className="bg-aqua text-navy hover:bg-aqua/90"
          onClick={() => window.location.href = `/checkout?dao=${dao.daoId}`}
        >
          Get Full Report — €49
        </Button>
        <Button
          variant="outline"
          className="border-aqua text-aqua hover:bg-aqua/10"
          onClick={() => window.open(`https://twitter.com/intent/tweet?text=${encodeURIComponent(
            `${dao.daoName} scores ${dao.overallScore.toFixed(1)}/10 on governance health. ${dao.teaser}. See full rankings: chainsights.one/rankings via @ChainSights`
          )}`, '_blank')}
        >
          Share on 𝕏
        </Button>
      </div>
    </div>
  )
}
```

---

## Integration Points

### 1. Free Tier System

**Integration:** Rankings read from `freeQueryLog` table

**Touchpoints:**
- User completes free rating → `publicListing` checkbox appears
- If checked → DAO becomes eligible for next weekly ranking
- `src/app/api/reports/free-rating/route.ts` already handles `publicListing` field

**No code changes needed** - integration point already exists.

---

### 2. Paid Reports

**Integration:** "Get Full Report" button links to existing checkout flow

**Flow:**
1. User clicks "Get Full Report — €49"
2. Redirects to `/checkout?dao={daoId}&tier=deep_dive`
3. Existing checkout page handles payment
4. Existing pipeline generates report

**No code changes needed** - reuse existing checkout.

---

### 3. Snapshot API

**Integration:** Reuse existing Snapshot client

**Current usage:**
- `src/lib/snapshot/client.ts` already fetches governance data
- Used by free tier and paid reports
- Rankings use data **already stored** in DB (no additional API calls)

**Performance impact:** None - rankings read from cache, not Snapshot API.

---

### 4. AI Analysis

**Integration:** Use existing Claude SDK integration

**For MVP (Phase 1):**
- Rankings use simple scores from `freeQueryLog.ratingScore`
- No additional AI calls needed

**For Enhanced (Phase 2):**
- Read `reports.aiAnalysis` JSON for detailed metrics
- Extract proposal quality, transparency, execution scores
- Fall back to simple scores if report unavailable

---

### 5. LogSnag Monitoring

**Integration:** Send events for ranking generation

**Events to track:**
```typescript
// Success
await log({
  channel: 'rankings',
  event: 'Weekly Rankings Generated',
  description: `${daosRanked} DAOs ranked for week ${snapshotWeek}`,
  icon: '📊',
  notify: false
})

// Failure
await log({
  channel: 'rankings',
  event: 'Ranking Generation Failed',
  description: error.message,
  icon: '❌',
  notify: true // Alert team
})

// Opt-out
await log({
  channel: 'rankings',
  event: 'DAO Opted Out',
  description: `${daoId} removed from public rankings`,
  icon: '🚫',
  notify: false
})
```

---

## Security Considerations

### 1. Rate Limiting

**Endpoints requiring rate limiting:**
- `POST /api/rankings/opt-out` → 10 requests/minute per IP
- `GET /api/rankings` → Public but cached (1 hour)

**Implementation:** Use Vercel Edge Config or Upstash Redis

```typescript
// src/lib/rate-limit.ts
import { Ratelimit } from '@upstash/ratelimit'
import { Redis } from '@upstash/redis'

const redis = new Redis({
  url: process.env.UPSTASH_REDIS_REST_URL!,
  token: process.env.UPSTASH_REDIS_REST_TOKEN!,
})

export const optOutRateLimit = new Ratelimit({
  redis,
  limiter: Ratelimit.slidingWindow(10, '1 m'),
})
```

---

### 2. Input Validation

**Opt-out endpoint:**
- Validate `daoId` format (alphanumeric + dots/hyphens)
- Check DAO exists in database
- Sanitize inputs to prevent SQL injection (Drizzle handles this)

```typescript
function validateDaoId(daoId: string): boolean {
  return /^[a-zA-Z0-9.-]+$/.test(daoId) && daoId.length < 100
}
```

---

### 3. QStash Signature Verification

**Cron job authentication:**
```typescript
import { verifySignature } from '@upstash/qstash/nextjs'

async function handler(req: NextRequest) {
  // Handler logic
}

export const POST = verifySignature(handler)
```

**Never** expose `/api/jobs/generate-rankings` publicly - always verify QStash signature.

---

### 4. GDPR Compliance

**Data handling:**
- `publicListing` flag = explicit consent to display DAO publicly
- Opt-out removes DAO from future rankings (past snapshots remain for historical integrity)
- No personal data collected (only DAO identifiers)

**Privacy policy disclosure:**
- Update `/privacy` page to mention public rankings
- Explain opt-in/opt-out process
- Note: Historical snapshots preserved for integrity (DAO name only, no PII)

---

### 5. Controversy Protection

**Legal disclaimers:**
- Display disclaimer on rankings page (already in methodology)
- Frame as "opinions" not "facts"
- Emphasize educational purpose
- Provide opt-out mechanism

**Already implemented** in methodology page.

---

## Performance & Caching

### 1. API Response Caching

**Strategy:** In-memory cache + HTTP cache headers

```typescript
// 1-hour in-memory cache
let cachedRankings: any = null
let cacheTimestamp = 0
const CACHE_TTL = 60 * 60 * 1000

// HTTP cache headers
return NextResponse.json(data, {
  headers: {
    'Cache-Control': 'public, max-age=3600', // 1 hour
    'Vary': 'Accept-Encoding'
  }
})
```

**Invalidation:**
- Automatic: After 1 hour
- Manual: When new snapshot generated (set `cachedRankings = null`)

---

### 2. Database Query Optimization

**Indexes:**
```sql
CREATE INDEX idx_ranking_snapshots_week ON ranking_snapshots(snapshot_week DESC);
CREATE INDEX idx_ranking_snapshots_dao ON ranking_snapshots(dao_id);
CREATE INDEX idx_freeQueryLog_publicListing ON freeQueryLog(public_listing) WHERE public_listing = true;
```

**Query patterns:**
- Fetch latest snapshot: `WHERE snapshot_week = '2025-W51'` (indexed)
- Fetch DAO history: `WHERE dao_id = 'ens.eth' ORDER BY snapshot_week DESC` (indexed)
- Eligible DAOs: `WHERE public_listing = true` (partial index)

---

### 3. Frontend Performance

**Strategies:**
- Server-side rendering (SSR) for initial page load
- Client-side expansion (no API calls for expand/collapse)
- Lazy load expanded details (only render when visible)
- Virtualization for 100+ DAOs (use `react-window` if needed)

**Current scale:** 20-50 DAOs → No optimization needed
**Future scale:** 100+ DAOs → Add virtualization

---

### 4. Calculation Job Performance

**Optimization:**
- Batch database queries (fetch all eligible DAOs in one query)
- Parallel score calculation (use `Promise.all()`)
- Single bulk insert for snapshots
- Timeout: 5 minutes (sufficient for 100 DAOs)

**Estimated processing time:**
- 20 DAOs: ~3 seconds
- 50 DAOs: ~8 seconds
- 100 DAOs: ~20 seconds

---

## Deployment Strategy

### Phase 1: MVP (Week 1)

**Goal:** Launch minimal leaderboard with manual data

**Implementation:**
1. Add `ranking_snapshots` table (migration)
2. Build calculation engine (`src/lib/rankings/`)
3. Create `/api/rankings` endpoint
4. Build frontend `/rankings` page
5. Manual first snapshot (run script locally)

**Timeline:** 6-8 hours
**Risk:** Low - no automation dependencies

---

### Phase 2: Automation (Week 2)

**Goal:** Add weekly cron job

**Implementation:**
1. Build `/api/jobs/generate-rankings` endpoint
2. Add QStash signature verification
3. Configure QStash cron schedule
4. Test with staging cron (daily for 1 week)
5. Switch to weekly schedule

**Timeline:** 3-4 hours
**Risk:** Medium - depends on QStash reliability

---

### Phase 3: Enhancement (Week 3+)

**Goal:** Improve scoring with detailed metrics

**Implementation:**
1. Extract metrics from `reports.aiAnalysis`
2. Add proposal quality, transparency, execution calculations
3. Improve teaser generation
4. Add historical trend charts
5. Add category filters

**Timeline:** 8-12 hours
**Risk:** Low - incremental improvements

---

### Rollback Plan

**If rankings cause controversy:**
1. Feature flag: `ENABLE_RANKINGS=false` in environment
2. Redirect `/rankings` to methodology page
3. Keep data in DB (don't delete)
4. Re-enable after adjusting methodology

**Implementation:**
```typescript
// src/app/rankings/page.tsx
if (process.env.ENABLE_RANKINGS !== 'true') {
  redirect('/rankings/methodology')
}
```

---

## Testing Strategy

### 1. Unit Tests

**Test files:**
- `src/lib/rankings/calculator.test.ts` (scoring algorithm)
- `src/lib/rankings/snapshot.test.ts` (snapshot generation)

**Test cases:**
```typescript
describe('calculateScores', () => {
  it('calculates correct overall score with weights', () => {
    const result = calculateScores({
      participation: 8,
      decentralization: 7,
      proposalQuality: 6,
      transparency: 9,
      execution: 5
    })
    expect(result.overall).toBeCloseTo(7.15) // Weighted average
  })

  it('handles missing metrics gracefully', () => {
    const result = calculateScores({
      ratingScore: 7.0,
      giniCoefficient: null,
      nakamotoCoefficient: null
    })
    expect(result.overall).toBeGreaterThan(0)
  })
})
```

---

### 2. Integration Tests

**Test scenarios:**
- Ranking generation with real database
- API endpoint returns valid JSON
- Opt-out updates correct records
- Cache invalidation works

**Run:** `npm run test:integration`

---

### 3. E2E Tests (Playwright)

**Test flows:**
```typescript
// tests/e2e/rankings.spec.ts
test('display leaderboard', async ({ page }) => {
  await page.goto('/rankings')
  await expect(page.locator('h1')).toContainText('Leaderboard')

  // Check table exists
  const rows = page.locator('tbody tr')
  await expect(rows).toHaveCount.greaterThan(0)
})

test('expand row shows details', async ({ page }) => {
  await page.goto('/rankings')
  await page.locator('tbody tr').first().click()

  // Check expanded content visible
  await expect(page.locator('text=Gini Coefficient')).toBeVisible()
})
```

---

### 4. Manual QA Checklist

**Before launch:**
- [ ] Rankings display correctly on mobile
- [ ] Score bars render with correct colors
- [ ] Delta indicators show +/- correctly
- [ ] "Get Report" button links to checkout
- [ ] "Share on 𝕏" opens Twitter intent
- [ ] Opt-out form works
- [ ] Methodology page loads
- [ ] Cron job runs successfully in staging
- [ ] Cache invalidates after new snapshot

---

## Migration Path

### Database Migration

**File:** `drizzle/migrations/XXXX_add_ranking_snapshots.sql`

```sql
-- Create ranking_snapshots table
CREATE TABLE ranking_snapshots (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

  snapshot_week TEXT NOT NULL,
  snapshot_date DATE NOT NULL,
  created_at TIMESTAMP DEFAULT NOW() NOT NULL,

  dao_id TEXT NOT NULL,
  dao_name TEXT NOT NULL,

  rank INTEGER NOT NULL,
  rank_previous INTEGER,

  overall_score REAL NOT NULL,
  participation_score REAL,
  decentralization_score REAL,
  proposal_quality_score REAL,
  transparency_score REAL,
  execution_score REAL,

  gini_coefficient REAL,
  nakamoto_coefficient INTEGER,
  voter_turnout_percentage REAL,
  unique_voters INTEGER,
  top5_percentage REAL,
  top10_percentage REAL,

  total_daos_ranked INTEGER NOT NULL,

  CONSTRAINT unique_dao_per_week UNIQUE (dao_id, snapshot_week)
);

CREATE INDEX idx_ranking_snapshots_week ON ranking_snapshots(snapshot_week DESC);
CREATE INDEX idx_ranking_snapshots_dao ON ranking_snapshots(dao_id);
CREATE INDEX idx_ranking_snapshots_rank ON ranking_snapshots(snapshot_week, rank);

-- Add index to freeQueryLog for performance
CREATE INDEX idx_freeQueryLog_publicListing
  ON free_query_log(public_listing)
  WHERE public_listing = true;
```

**Run migration:**
```bash
npm run db:generate
npm run db:push
```

---

### Backward Compatibility

**Considerations:**
- `freeQueryLog` already has `publicListing` field (no migration needed)
- Opt-out endpoint already implemented
- No breaking changes to existing features

**Safe to deploy:** Yes - additive only.

---

## Appendix

### Key Files Summary

| File | Purpose |
|------|---------|
| `src/lib/db/schema.ts` | Add `rankingSnapshots` table definition |
| `src/lib/rankings/calculator.ts` | Scoring algorithm implementation |
| `src/lib/rankings/snapshot.ts` | Snapshot generation & retrieval |
| `src/lib/rankings/types.ts` | TypeScript interfaces |
| `src/app/api/rankings/route.ts` | Public API endpoint |
| `src/app/api/rankings/history/route.ts` | Historical data API |
| `src/app/api/jobs/generate-rankings/route.ts` | Weekly cron job |
| `src/app/rankings/page.tsx` | Public leaderboard page |
| `src/components/rankings/RankingRow.tsx` | Leaderboard row component |
| `src/components/rankings/ScoreBar.tsx` | Visual score indicator |

---

### Environment Variables

```bash
# Existing (no changes)
DATABASE_URL=postgresql://...
QSTASH_TOKEN=...
QSTASH_CURRENT_SIGNING_KEY=...
QSTASH_NEXT_SIGNING_KEY=...

# Optional: Feature flag
ENABLE_RANKINGS=true
```

---

### Dependencies

**No new dependencies required** - reuse existing stack:
- Next.js 14
- Drizzle ORM
- QStash SDK
- PostgreSQL (Neon)

---

## Sign-off

**Architects:**
- Sally (Solutions Architect) - Approved
- Winston (Tech Lead) - Approved

**Ready for:** Epics & Stories breakdown

**Next steps:**
1. Review with Mario
2. Create implementation epics
3. Assign to development team

---

**Document version:** 1.0
**Last updated:** December 21, 2025
**Status:** Ready for Implementation
