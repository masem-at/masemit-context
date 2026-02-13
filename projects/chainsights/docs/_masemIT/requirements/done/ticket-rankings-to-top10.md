# DEV TICKET: Rankings → "Top 10 DAOs" Rename & Simplification

**Priority:** Medium (Teil des Open Matrix Epic)
**Effort:** Small
**Dependencies:** Open DAO Matrix Ticket

---

## Context

Mit dem Freemium-Pivot (alle Daten offen in der Matrix) wird die Rankings-Page in ihrer jetzigen Form redundant. Die Matrix bietet dieselben Daten plus Filter, Sortierung und Suche.

Statt Rankings zu entfernen, positionieren wir sie als **einfachen Einstieg für Casual Visitors** — "Top 10 DAOs" ist sofort verständlich und weniger overwhelming als die volle Matrix.

---

## Änderungen

### 1. Rename

| Alt | Neu |
|-----|-----|
| "Rankings" | "Top 10 DAOs" |
| URL: `/rankings` | URL: `/top-daos` (mit Redirect von `/rankings`) |
| Nav Label: "Rankings" | Nav Label: "Top 10" oder "Top DAOs" |

### 2. Paywall entfernen

Aktuell hat Rankings auch Free/Email/Paid Einschränkungen — diese komplett entfernen. Alle 10 DAOs sichtbar ohne Login.

### 3. Content-Anpassung

**Page Title:** "Top 10 DAOs by Governance Health"

**Subtitle:** "The highest-scoring DAOs in the Decentralized Governance Index"

**Anzeige:** Nur Top 10, nicht mehr. Clean und focused.

**CTA am Ende:**
```
Want to see all 46 DAOs? 
[View Full DAO Matrix →]
```

### 4. Redirects

**Rankings Redirect:**
`/rankings` → 301 Redirect zu `/top-daos`

**Vanity URLs für alle DAOs:**
`/radiant-capital` → 301 Redirect zu `/matrix/radiant-capital`
`/ens` → 301 Redirect zu `/matrix/ens`
`/lido` → 301 Redirect zu `/matrix/lido`
... etc. für alle 46 DAOs

**Warum Vanity URLs:**
- Kürzer und cleaner für Social Sharing
- "chainsights.one/radiant-capital" vs "chainsights.one/matrix/radiant-capital"
- Fühlt sich an wie "die DAO hat eine eigene Seite bei uns"
- Leichter zu kommunizieren in Discord/DMs

**Implementierung:**
Entweder statisch in `next.config.js` oder dynamisch via Middleware die alle bekannten DAO-Slugs matcht und weiterleitet.

---

## User Journey (neu)

```
Top 10 DAOs (Einstieg, schneller Überblick)
    ↓
DAO Matrix (alle 46, Filter, Deep Dive)
    ↓
DAO Detail Page (einzelne DAO mit Charts)
    ↓
Report CTA (Free Check / Deep Dive / Audit)
```

---

## Navigation Update

**Header Nav (vorher):**
```
Rankings | Governance Index | DAO Matrix | Pricing
```

**Header Nav (nachher):**
```
Top 10 | Governance Index | DAO Matrix | Pricing
```

Oder alternativ:
```
Top DAOs | DAO Matrix | Methodology | Pricing
```

(Governance Index → "Methodology" wäre auch klarer, aber separates Ticket)

---

## Analytics Events

Bestehende Events anpassen:

```typescript
// ALT
track("rankings_view", { ... });

// NEU
track("top_daos_view", { ... });
```

Oder beides feuern für Übergangszeit:
```typescript
track("top_daos_view", { ... });
track("rankings_view", { ... }); // deprecated, für Vergleichbarkeit
```

---

## Acceptance Criteria

- [ ] Page renamed zu "Top 10 DAOs"
- [ ] URL geändert zu `/top-daos`
- [ ] 301 Redirect von `/rankings` zu `/top-daos`
- [ ] Vanity URLs für alle 46 DAOs (`/{dao-slug}` → `/matrix/{dao-slug}`)
- [ ] Alle Paywall/Login-Gates entfernt
- [ ] Nur Top 10 angezeigt (nicht mehr)
- [ ] CTA zu DAO Matrix am Seitenende
- [ ] Nav Label aktualisiert
- [ ] Analytics Event umbenannt oder erweitert
- [ ] OG Meta Tags aktualisiert

---

## Files to Touch

1. Rankings Page → Rename, URL ändern
2. Navigation Component — Label update
3. next.config.js oder middleware — Redirects (Rankings + alle DAO Vanity URLs)
4. Analytics Events — rename

---

## Out of Scope

- "Governance Index" → "Methodology" Rename (separates Ticket falls gewünscht)
- Redesign der Top 10 Seite
- Kategorie-spezifische Top 10 (z.B. "Top 10 DeFi DAOs")
