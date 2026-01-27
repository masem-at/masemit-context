# Feature Spec: Governance Health Check Quiz

**Date:** 2025-12-17
**Authors:** Mary (Analyst), Sally (UX), John (PM), Amelia (Dev), Murat (TEA)
**Status:** Ready for Implementation
**Priority:** High

---

## 1. Overview

An interactive 5-question quiz that helps DAO decision-makers assess their governance visibility. Positioned as a lead generation and education tool that funnels users to either the full guide or a report purchase.

**URL:** `/quiz` (or `/governance-health-check`)

---

## 2. User Journey

```
Landing/Home → [Take the Quiz] → 5 Questions → Score + Results → Guide or Purchase
```

**Entry points:**
- Homepage CTA
- Navigation menu
- End of guide page
- Social media links

---

## 3. Quiz Questions & Scoring

### Question 1: Power Concentration
> **"Do you know what percentage of voting power your top 10 wallets control?"**

| Answer | Score | Insight |
|--------|-------|---------|
| Yes, I know the exact percentage | 2 | ✅ Power concentration tracked |
| I have a rough idea | 1 | ⚠️ Approximate awareness |
| No idea | 0 | ❌ Power concentration unknown |
| We don't track this | 0 | ❌ Power concentration unknown |

### Question 2: Voter Classification
> **"Can you distinguish between whales (large token holders) and delegates (trusted with others' votes) in your governance?"**

| Answer | Score | Insight |
|--------|-------|---------|
| Yes, we track and distinguish both | 2 | ✅ Voter classification clear |
| We know who the big voters are, but don't classify them | 1 | ⚠️ Partial classification |
| No, votes are votes to us | 0 | ❌ Whale vs delegate unknown |
| I'm not sure what the difference is | 0 | ❌ Whale vs delegate unknown |

### Question 3: Participation Trends
> **"Do you know if voter participation is trending up or down compared to last quarter?"**

| Answer | Score | Insight |
|--------|-------|---------|
| Yes, I can cite the specific change | 2 | ✅ Participation trend tracked |
| I have a general sense of the direction | 1 | ⚠️ Approximate trend awareness |
| No, we don't track trends | 0 | ❌ Participation trend unknown |
| We've never compared periods | 0 | ❌ Participation trend unknown |

### Question 4: Technical Metrics
> **"Do you know your DAO's Gini coefficient for voting power distribution?"**

| Answer | Score | Insight |
|--------|-------|---------|
| Yes, I know our current Gini | 2 | ✅ Technical metrics tracked |
| I know what it is but not our number | 1 | ⚠️ Concept known, data missing |
| I've heard of it but don't use it | 0 | ❌ Gini coefficient unknown |
| What's a Gini coefficient? | 0 | ❌ Gini coefficient unknown |

### Question 5: Review Recency
> **"When did you or your team last conduct a governance health review?"**

| Answer | Score | Insight |
|--------|-------|---------|
| Within the last 30 days | 2 | ✅ Recent review conducted |
| Within the last 90 days | 1 | ⚠️ Moderately recent |
| More than 90 days ago | 0 | ❌ Review overdue |
| We've never done one | 0 | ❌ Review overdue |

---

## 4. Scoring & Results

### Score Calculation
- **Maximum:** 10 points (5 questions × 2 points each)
- **Minimum:** 0 points

### Score Brackets

| Score | Label | Emoji | Color | Message |
|-------|-------|-------|-------|---------|
| 9-10 | Governance Expert | 🏆 | Green | "Impressive! You have strong visibility into your DAO's governance health." |
| 6-8 | Informed with Gaps | 📊 | Aqua | "Good foundation, but some blind spots could be costing you." |
| 3-5 | Limited Visibility | ⚠️ | Yellow | "You're missing critical insights that affect governance decisions." |
| 0-2 | Flying Blind | 🚨 | Red | "Your governance visibility is critically low. Time to change that." |

### Results Display

```
┌─────────────────────────────────────────────────────────────┐
│  YOUR GOVERNANCE VISIBILITY SCORE                           │
│                                                             │
│              6 / 10                                         │
│         ████████████░░░░░░░░                                │
│                                                             │
│         📊 Informed with Gaps                               │
│                                                             │
│  "Good foundation, but some blind spots could be            │
│   costing you."                                             │
│                                                             │
├─────────────────────────────────────────────────────────────┤
│  YOUR BREAKDOWN                                             │
│                                                             │
│  ✅ Power concentration tracked                              │
│  ✅ Participation trend tracked                              │
│  ⚠️ Partial voter classification                            │
│  ❌ Gini coefficient unknown                                 │
│  ❌ Review overdue                                           │
│                                                             │
├─────────────────────────────────────────────────────────────┤
│  RECOMMENDED NEXT STEPS                                     │
│                                                             │
│  📖 Read our guide to understand what you're missing        │
│     [Read the Guide →]                                      │
│                                                             │
│  📊 Get a professional report for your DAO                  │
│     [Order a Governance Report →]                           │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 5. CTA Strategy by Score

| Score | Primary CTA | Secondary CTA |
|-------|-------------|---------------|
| 9-10 | "Validate with a professional report" → Order | "Share your expertise" → Twitter |
| 6-8 | "Fill the gaps" → Order | "Learn what you're missing" → Guide |
| 3-5 | "Start with our guide" → Guide | "Or get expert help" → Order |
| 0-2 | "Learn the basics" → Guide | "Let us help" → Order |

---

## 6. Optional Email Capture

**Placement:** After score reveal, before full breakdown

**Copy:**
> "Get your personalized governance checklist sent to your inbox"
> [Email] [Send My Results →]
> _No spam. Unsubscribe anytime._

**Behavior:**
- Optional (can skip)
- If provided: sends results + checklist PDF + links to guide
- Stores lead in database for follow-up

---

## 7. UI/UX Specifications

### Layout
- **Desktop:** Centered card (max-width 640px), progress bar at top
- **Mobile:** Full-width, stacked answer cards, large tap targets (min 48px)

### Colors (using brand palette)
- Background: Navy Dark (`#0A1F3D`)
- Card: Navy (`#0F2C55`)
- Progress/Accent: Aqua (`#00E5C0`)
- Text: White/Gray
- Score colors per bracket (see Section 4)

### Animations
- Question transition: Fade + slight slide
- Progress bar: Smooth fill
- Score reveal: Count-up animation (0 → final score)
- Checkmarks/X: Pop-in with slight bounce

### Components

```
<QuizPage>
  ├── <QuizHero> (intro + start button)
  ├── <QuizContainer>
  │   ├── <ProgressBar current={n} total={5} />
  │   ├── <QuestionCard
  │   │     question={...}
  │   │     answers={[...]}
  │   │     onAnswer={(score) => ...}
  │   │   />
  │   └── <NavigationButtons />
  ├── <ResultsCard>
  │   ├── <ScoreDisplay score={n} bracket={...} />
  │   ├── <BreakdownList insights={[...]} />
  │   ├── <EmailCapture optional />
  │   └── <CTAButtons primary={...} secondary={...} />
  └── <GuideTeaser> (below results)
</QuizPage>
```

---

## 8. Analytics Events

| Event | Trigger | Properties |
|-------|---------|------------|
| `quiz_page_viewed` | Page load | `source` (referrer) |
| `quiz_started` | Click "Start" | — |
| `quiz_question_answered` | Select answer | `question_number`, `score`, `answer_index` |
| `quiz_completed` | All 5 answered | `total_score`, `bracket`, `answers[]` |
| `quiz_email_submitted` | Email captured | `score`, `bracket` |
| `quiz_cta_clicked` | CTA click | `cta_type` (guide/order), `score` |

---

## 9. Technical Implementation

### Route
`/quiz` → `src/app/quiz/page.tsx`

### State Management
```typescript
interface QuizState {
  status: 'intro' | 'in_progress' | 'completed'
  currentQuestion: number // 0-4
  answers: (number | null)[] // [2, 1, 0, null, null]
  email?: string
}
```

### Data Structure
```typescript
interface Question {
  id: string
  text: string
  answers: {
    text: string
    score: 0 | 1 | 2
    insight: string
    insightType: 'positive' | 'warning' | 'negative'
  }[]
}

interface ScoreBracket {
  min: number
  max: number
  label: string
  emoji: string
  color: string
  message: string
  primaryCTA: { text: string; href: string }
  secondaryCTA: { text: string; href: string }
}
```

### Storage
- `localStorage`: Save progress for returning visitors
- Optional: `/api/quiz-results` POST for analytics

---

## 10. Content for Guide Page

After quiz, link to `/guide` with the full "How to Read a Governance Report" content.

**Guide page structure:**
1. Hero with score reminder (if came from quiz)
2. 5 Core Questions (visual cards)
3. Key Metrics Explained (accordion sections)
4. Red Flags
5. Checklist
6. Case Example
7. CTA: Order a Report

---

## 11. Acceptance Criteria

- [ ] Quiz loads at `/quiz`
- [ ] 5 questions display sequentially with progress bar
- [ ] Answers are clickable cards with hover/selected states
- [ ] Score calculates correctly (0-10)
- [ ] Results show correct bracket, message, and breakdown
- [ ] CTAs link to guide and order pages
- [ ] Mobile responsive (stacked cards, large tap targets)
- [ ] Analytics events fire correctly
- [ ] Optional email capture works
- [ ] Progress saves to localStorage
- [ ] Animations are smooth (no jank)

---

*Spec created by the ChainSights Party Mode team — 2025-12-17*
