# DEV TICKET: Signal Scanner — Relevance Scoring & Prioritization

**Priority:** High
**Effort:** Medium
**Dependencies:** Signal Scanner Phase 1 (bereits aktiv)

---

## Implementation Status

### Phase 1 — ✅ COMPLETE (2026-02-05)

| Feature | Status | Notes |
|---------|--------|-------|
| Keyword-based Topic Scoring | ✅ | `scoring-config.ts` with configurable keyword lists |
| DAO Priority from Engagement Hub | ✅ | Maps engagement_status → score (active=5, warm=4, etc.) |
| Freshness scoring | ✅ | Age-based (<6h=5, 6-24h=4, 1-3d=3, etc.) |
| 3-Tier UI (Hot/Relevant/Low) | ✅ | Hot ≥4.0, Relevant 3.0-3.9, Low <3.0 |
| Keywords in Config file | ✅ | `src/lib/signals/scoring-config.ts` |
| Default: Show only Hot | ✅ | Filter tabs with counts |

**Commits:**
- `be420ba` feat(signals): add relevance scoring with 3-tier UI (Phase 3b.1)

**Files:**
- `src/lib/signals/scoring-config.ts` — Keyword lists, weights, thresholds
- `src/lib/signals/scoring.ts` — Score calculation functions
- `src/lib/signals/forum-scanner.ts` — Updated to calculate scores
- `src/app/admin/engagement/engagement-hub-client.tsx` — 3-tier UI

**Gewichtung (angepasst ohne Reply Count):**
- Topic: 50% (statt 40%)
- DAO Priority: 30% (statt 25%)
- Freshness: 20% (statt 15%)

### Phase 2 — BACKLOG

- [ ] Reply-Count Fetch (Discourse JSON API)
- [ ] Score-Breakdown Logging für Debugging
- [ ] Optional: AI für Grenzfälle

---

## Context

Der Governance Signal Scanner funktioniert — zu gut. 6 DAOs mit Forum generieren bereits 142 Signale an einem Tag. Hochgerechnet auf alle DAOs mit Forum: ~1.000 Signale täglich.

Das ist unmöglich manuell abzuarbeiten. Wir brauchen intelligente Priorisierung.

**Ziel:** Aus 1.000 Signalen die 5-10 identifizieren die tatsächlich Engagement-würdig sind.

---

## Lösung: Relevanz-Score

Jedes Signal bekommt einen **Relevanz-Score von 1-5** basierend auf mehreren Faktoren.

### Scoring-Faktoren

| Faktor | Gewichtung (Phase 1) | Gewichtung (Phase 2) | Beschreibung |
|--------|---------------------|----------------------|--------------|
| **Topic Relevance** | **50%** | 40% | Wie relevant ist das Thema für ChainSights? |
| **DAO Priority** | **30%** | 25% | Ist das ein DAO wo wir aktiv werden wollen? |
| **Engagement Potential** | — | 20% | Hat der Thread Traction / Diskussion? (Phase 2) |
| **Freshness** | **20%** | 15% | Wie neu ist das Signal? |

### Topic Relevance (1-5)

**Score 5 — Bullseye:**
Keywords: `delegation`, `delegate`, `voting power`, `whale`, `concentration`, `governance health`, `participation`, `decentralization`, `identity`, `sybil`

**Score 4 — Sehr relevant:**
Keywords: `governance`, `voting`, `quorum`, `proposal threshold`, `delegate compensation`

**Score 3 — Relevant:**
Tags: `frustration`, `delegation`, `question` über Governance-Struktur

**Score 2 — Möglicherweise relevant:**
Tags: `proposal` (generisch)

**Score 1 — Wahrscheinlich nicht relevant:**
Rein technische Proposals (Risk Parameters, Supply Caps, Protocol Upgrades ohne Governance-Bezug)

### DAO Priority (1-5)

Basierend auf Engagement Hub Status:

| Status | Score | Beispiele |
|--------|-------|-----------|
| Active | 5 | DAOs wo wir aktiv engagen |
| Warm | 4 | Radiant, ENS, Giveth, Lido |
| Pending | 3 | Kontakt hergestellt, warten auf Response |
| Target | 2 | Auf der Liste, aber noch nicht kontaktiert |
| Not Started | 1 | Kein Plan für Engagement |

**Oder:** DGI Top 10 = Score 4-5, Rest absteigend nach Rang.

### Engagement Potential (1-5)

| Indikator | Score |
|-----------|-------|
| Viele Replies (>10) | 5 |
| Einige Replies (5-10) | 4 |
| Wenige Replies (2-5) | 3 |
| Keine Replies, aber Views | 2 |
| Frisch gepostet, noch keine Reaktion | 3 (Chance früh dabei zu sein) |

### Freshness (1-5)

| Alter | Score |
|-------|-------|
| < 6 Stunden | 5 |
| 6-24 Stunden | 4 |
| 1-3 Tage | 3 |
| 3-7 Tage | 2 |
| > 7 Tage | 1 |

---

## Berechnung

```
Relevanz-Score = (Topic × 0.4) + (DAO Priority × 0.25) + (Engagement × 0.2) + (Freshness × 0.15)
```

**Beispiel — Aave Delegation Thread:**
- Topic: "delegation" → 5
- DAO Priority: Aave ist Top 10 DGI → 4
- Engagement: 8 Replies → 4
- Freshness: Heute gepostet → 5

Score = (5 × 0.4) + (4 × 0.25) + (4 × 0.2) + (5 × 0.15) = 2.0 + 1.0 + 0.8 + 0.75 = **4.55 / 5**

→ High Priority, sofort anschauen!

**Beispiel — Aave Risk Parameter Update:**
- Topic: "Supply/Borrow Caps" → 1
- DAO Priority: Aave → 4
- Engagement: 2 Replies → 3
- Freshness: Heute → 5

Score = (1 × 0.4) + (4 × 0.25) + (3 × 0.2) + (5 × 0.15) = 0.4 + 1.0 + 0.6 + 0.75 = **2.75 / 5**

→ Low Priority, ignorieren.

---

## UI Updates

### Signal List View

**Aktuell:**
```
🟢 Aave DAO [frustration]
Unlocking Yield: Aave V4's Reinvestment Controller
by LlamaRisk · 5.2.2026
```

**Neu:**
```
⭐⭐⭐⭐⭐ 🟢 Aave DAO [delegation]
AL Development Update | January 2026
by AaveLabs · 5.2.2026 · 12 replies
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

⭐⭐⭐ 🟢 Aave DAO [proposal]
Chaos Labs Risk Stewards - Increase Supply...
by ChaosLabs · 5.2.2026 · 3 replies
```

Oder mit numerischem Score:
```
[4.8] 🟢 Aave DAO [delegation] ...
[2.3] 🟢 Aave DAO [proposal] ...
```

### Filter & Sort

**Filter-Optionen:**
- Minimum Score: [3+] [4+] [5 only]
- Tags: [delegation] [frustration] [question] [proposal]
- DAO Status: [Active] [Warm] [All]
- Timeframe: [Today] [This Week] [All]

**Default Sort:** Score (highest first)

### Daily Digest View (optional)

Eine "Top 10 Today" Ansicht die nur die höchsten Scores zeigt:

```
📬 Today's Top Signals (10 of 142)

1. [4.8] Aave — Delegation Framework Discussion
2. [4.6] ENS — Delegate Compensation Review
3. [4.5] Lido — Voting Power Concentration Concerns
...
```

---

## Notifications (Future Enhancement)

Optional: Push/Email für Signale mit Score 4.5+

```
🔔 High-Priority Signal Detected

Aave DAO — "Mandatory Disclosures and Conflict-of-Interest Voting Norms"
Score: 4.7/5 | Tag: delegation | 8 replies

[View Signal] [Mark as Read]
```

---

## Analytics Events

```typescript
// Signal interactions
track("signal_view", {
  dao: "aave",
  score: 4.8,
  tag: "delegation",
  action: "expanded"
});

track("signal_engage", {
  dao: "aave",
  score: 4.8,
  tag: "delegation",
  action: "clicked_forum_link"
});

track("signal_dismiss", {
  dao: "aave", 
  score: 2.3,
  tag: "proposal",
  reason: "not_relevant"  // optional feedback
});
```

---

## Acceptance Criteria

- [ ] Jedes Signal hat einen Relevanz-Score (1-5, eine Dezimalstelle)
- [ ] Score wird basierend auf Topic, DAO Priority, Engagement, Freshness berechnet
- [ ] Signal-Liste ist standardmäßig nach Score sortiert (höchster zuerst)
- [ ] Filter für Minimum Score verfügbar
- [ ] Filter für Tags verfügbar
- [ ] Filter für DAO Status verfügbar (aus Engagement Hub)
- [ ] Score ist visuell sichtbar (Sterne oder Nummer)
- [ ] Reply-Count wird angezeigt (wenn verfügbar vom Scraper)
- [ ] "Top 10 Today" Quick-View (optional, nice-to-have)
- [ ] Analytics Events für Signal-Interaktionen

---

## Technical Notes

### Topic Classification

Die AI macht bereits Tag-Klassifizierung (`frustration`, `proposal`, etc.). Erweiterung:

```typescript
// Zusätzlich zum Tag: Topic Keywords extrahieren
interface Signal {
  id: string;
  dao: string;
  title: string;
  url: string;
  tag: "frustration" | "proposal" | "delegation" | "question";
  
  // NEU
  topicKeywords: string[];  // ["delegation", "compensation", "voting power"]
  topicRelevanceScore: number;  // 1-5 von AI
  replyCount: number;
  postedAt: Date;
  relevanceScore: number;  // Final computed score
}
```

### DAO Priority Sync

DAO Priority kommt aus dem Engagement Hub. Entweder:
- Join bei der Query
- Oder Engagement Status in der Signal-Tabelle cachen

### Scoring Service

```typescript
function calculateRelevanceScore(signal: Signal, daoStatus: EngagementStatus): number {
  const topicScore = signal.topicRelevanceScore;  // From AI
  const daoScore = mapStatusToScore(daoStatus);   // Active=5, Warm=4, etc.
  const engagementScore = mapRepliesToScore(signal.replyCount);
  const freshnessScore = mapAgeToScore(signal.postedAt);
  
  return (topicScore * 0.4) + (daoScore * 0.25) + (engagementScore * 0.2) + (freshnessScore * 0.15);
}
```

---

## Out of Scope

- Automatisches Engagement (Auto-Reply) — bleibt manuell
- Push Notifications — Future Enhancement
- ML-basiertes Scoring (lernt aus Mario's Interaktionen) — Future Enhancement
- X/Twitter Signals — wartet auf API Key, separates Ticket
