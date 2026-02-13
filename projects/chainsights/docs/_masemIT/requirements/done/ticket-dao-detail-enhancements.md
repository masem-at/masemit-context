# DEV TICKET: DAO Detail Page Enhancements

**Priority:** Medium
**Effort:** Small
**Dependencies:** Open DAO Matrix Epic (sollte danach kommen)

---

## Context

Die DAO Detail Pages zeigen jetzt alle Charts offen — gut! Aber es fehlen drei Elemente die den "Share-Moment" und das Verständnis verbessern:

1. **Share Buttons** — Leute wollen ihren Score teilen, aber es gibt keinen einfachen Weg
2. **Rank Badge** — "#1 overall" fehlt komplett, dabei ist das der Pride-Moment
3. **Benchmark Context** — Ist 8.6 gut? Ohne Vergleich weiß man es nicht

---

## 1. Share Buttons

**Position:** Rechts unten auf der Page, oder im Header-Bereich neben dem Score

**Buttons:**
```
[📋 Copy Link]  [𝕏 Tweet]  [💬 Discord]
```

**Funktionalität:**

### Copy Link
- Kopiert die Vanity URL: `chainsights.one/radiant-capital`
- Tooltip/Toast: "Link copied!"

### Tweet (X)
Öffnet Twitter Intent mit vorausgefülltem Text:

```
https://twitter.com/intent/tweet?text={encoded_text}&url={url}
```

**Tweet Template:**
```
{DAO_NAME} scores {SCORE} ({GRADE}) in the @ChainSights_one Governance Index — #{RANK} of 46 DAOs.

Check your DAO 👇
```

**Beispiel für Radiant:**
```
Radiant Capital scores 8.6 (A) in the @ChainSights_one Governance Index — #1 of 46 DAOs.

Check your DAO 👇
chainsights.one/radiant-capital
```

### Discord (optional)
- Kopiert einen Discord-formatierten Text in die Zwischenablage
- Toast: "Copied for Discord!"

**Discord Template:**
```
**Radiant Capital** — DGI Score: 8.6 (A) | #1 of 46 DAOs
🔗 https://chainsights.one/radiant-capital
```

---

## 2. Rank Badge

**Position:** Im Score-Header rechts oben, unter oder neben "High confidence"

**Aktuell:**
```
┌─────────────────────────┐
│  8.6  │ — 0.00          │
│   A   │ ● High confidence│
│       │ Updated Feb 5    │
└─────────────────────────┘
```

**Neu:**
```
┌─────────────────────────────┐
│  8.6  │ — 0.00              │
│   A   │ ● High confidence   │
│       │ 🏆 #1 overall       │
│       │ 🥇 #1 in DeFi       │
│       │ Updated Feb 5       │
└─────────────────────────────┘
```

**Varianten je nach Rang:**

| Rang | Icon | Text |
|------|------|------|
| #1 | 🏆 oder 🥇 | #1 overall |
| #2 | 🥈 | #2 overall |
| #3 | 🥉 | #3 overall |
| #4-10 | 📊 | Top 10 (#4 overall) |
| #11+ | 📊 | #14 of 46 |

Plus Kategorie-Rang:
- "#1 in DeFi" / "#3 in Infrastructure" / "#2 in Public Goods"

**Datenquelle:** Rang kommt aus der Matrix-Sortierung, sollte bereits im Backend verfügbar sein.

---

## 3. Benchmark Context

**Position:** Direkt unter dem großen Score, oder als Subtitle

**Aktuell:**
```
Radiant Capital (DeFi)
radiantcapital.eth

8.6 A
```

**Neu:**
```
Radiant Capital (DeFi)
radiantcapital.eth

8.6 A
━━━━━━━━━━━━━━━━━━━━━
Top 3% of all DAOs · Ecosystem avg: 5.4
```

**Oder als visueller Balken:**
```
        Radiant
           ↓
[░░░░░░░░░░░░░░░░░▓▓▓]
0        avg 5.4     10
```

**Text-Varianten basierend auf Score:**

| Score Range | Percentile | Text |
|-------------|------------|------|
| 8.0+ | Top 10% | "Top 10% of all DAOs" |
| 7.0-7.9 | Top 25% | "Top 25% of all DAOs" |
| 5.5-6.9 | Average | "Around ecosystem average" |
| < 5.5 | Below avg | "Below ecosystem average (5.4)" |

**Berechnung:**
- Ecosystem Average: Durchschnitt aller 46 DGI Scores (aktuell ~5.4)
- Percentile: Position / Total DAOs

---

## Analytics Events

```typescript
// Share Button Clicks
track("dao_share_click", {
  dao: "radiant-capital",
  method: "copy_link" | "twitter" | "discord",
  score: 8.6,
  rank: 1
});

// Optional: Track wenn jemand den kopierten Link tatsächlich teilt
// (schwer zu messen, aber Twitter Intent Clicks sind ein Proxy)
```

---

## UI/UX Notes

**Share Buttons Styling:**
- Subtle, nicht zu prominent — die Daten sind der Star
- Hover-State zeigt Tooltip was passiert
- Nach Klick: kurzes Feedback (Toast/Checkmark)

**Rank Badge:**
- Gold/Gelb für #1, Silber für #2, Bronze für #3
- Restliche Ränge in neutraler Farbe
- Klickbar? Könnte zur Matrix mit Highlight scrollen

**Benchmark:**
- Grün wenn über Average, Rot wenn unter
- Oder neutral — wir wollen nicht shamen, nur kontextualisieren

---

## Mobile Considerations

- Share Buttons sollten auch auf Mobile gut erreichbar sein
- Rank Badge muss in den kompakteren Mobile-Header passen
- Benchmark-Text kann auf Mobile kürzer sein: "Top 3% · Avg: 5.4"

---

## Acceptance Criteria

- [ ] Share Buttons sichtbar auf DAO Detail Page
- [ ] Copy Link kopiert Vanity URL und zeigt Feedback
- [ ] Twitter/X öffnet Intent mit vorausgefülltem Tweet inkl. Score + Rang
- [ ] Discord kopiert formatierten Text
- [ ] Rank Badge zeigt "#X overall" und "#X in {Category}"
- [ ] Top 3 haben spezielle Icons (🥇🥈🥉)
- [ ] Benchmark-Text zeigt Percentile und Ecosystem Average
- [ ] Benchmark-Werte werden dynamisch berechnet
- [ ] `dao_share_click` Event feuert bei jedem Share-Button-Klick
- [ ] Mobile-responsive

---

## Files to Touch

1. DAO Detail Page Component — Share Buttons hinzufügen
2. DAO Detail Page Component — Rank Badge im Header
3. DAO Detail Page Component — Benchmark Context Section
4. Analytics — neues `dao_share_click` Event
5. Backend/API — Rang und Percentile mitliefern (falls nicht schon)

---

## Out of Scope

- Share to LinkedIn (kann später kommen)
- Share to Telegram (kann später kommen)
- Open Graph Image Generation per DAO (separates Ticket, wäre aber nice für Twitter Cards)
- "Compare with another DAO" Feature
