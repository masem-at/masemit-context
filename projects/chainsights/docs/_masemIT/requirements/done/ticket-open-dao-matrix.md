# DEV TICKET: Open DAO Matrix & Detail Pages (Freemium Pivot)

> **⚠️ PRICING OUTDATED** — DAO Matrix is FREE since 2026-02-05. €19/mo and €99/yr subscriptions are deprecated. See `docs/project_context.md`.

**Priority:** High
**Effort:** Small-Medium
**Dependencies:** None (frontend-only changes)

---

## Context

Die DAO Matrix und DAO Detail-Seiten sind aktuell hinter einer Paywall:
- Matrix zeigt 5 DAOs, dann "41 more DAOs hidden — Sign in to unlock"
- Detail-Seiten zeigen 1 Chart (GVS), alle anderen: "Sign in to unlock more charts"
- Eingeloggte (nicht zahlende) User sehen mehr, aber nicht alles
- Zahlende Subscriber sehen alles + CSV/JSON Export

**Problem:** Genau die Leute die unsere besten Multiplikatoren wären (DAO Delegates, Governance-Interessierte), werden vor einer Paywall geblockt. Radiant Capital hat bewiesen dass offene Daten geteilt werden (45 Referrals an einem Tag). Ein DAO-Delegate der seinen Score nicht sehen kann, teilt den Link nicht. Einer der seinen Score sieht und stolz drauf ist, teilt sofort.

In einem Forum-Post wurde die Paywall direkt kritisiert. Wir verlieren Vertrauen und Reichweite durch zu frühes Gating.

**Entscheidung:** Die Rohdaten werden komplett frei. Die Analyse/Interpretation kostet Geld.

---

## Neues Freemium-Modell

### Gratis für alle (kein Login, kein Account)

| Feature | Aktuell | Neu |
|---------|---------|-----|
| DAO Matrix — alle DAOs sichtbar | ❌ 5 sichtbar, Rest hidden | ✅ Alle 46 DAOs sichtbar |
| DAO Matrix — Sortierung + Filter | ❌ Nur für eingeloggte | ✅ Für alle |
| DAO Matrix — Suche | ✅ | ✅ |
| DAO Matrix — Kategorie-Filter | ✅ | ✅ |
| DAO Detail — GVS Chart | ✅ | ✅ |
| DAO Detail — HPR Chart | ❌ Locked | ✅ Offen |
| DAO Detail — DEI Chart | ❌ Locked | ✅ Offen |
| DAO Detail — PDI Chart | ❌ Locked | ✅ Offen |
| DAO Detail — GPI Chart | ❌ Locked | ✅ Offen |
| DAO Detail — Benchmark vs. Avg | ❌ Locked | ✅ Offen |
| DAO Detail — Trend (7d) | ✅ | ✅ |
| Rankings Page | ✅ | ✅ |
| Governance Index Page | ✅ | ✅ |

### Bezahlt — Matrix Subscription (€19/mo oder €99/yr)

| Feature | Beschreibung |
|---------|-------------|
| CSV Export | Download aller Scores als CSV |
| JSON Export | API-artiger JSON Download |
| Comparison Mode | Side-by-side DAO Vergleich (falls vorhanden/geplant) |
| Historische Daten | Zugriff auf Daten > 4 Wochen zurück |
| Priority Support | Schnellere Antworten |

### Bezahlt — One-Time Reports

| Tier | Preis | Was |
|------|-------|-----|
| Free Check | Gratis | GVS Score, 4 KPI Breakdown, Category Comparison |
| Deep Dive | €49 | + AI-Analyse, Empfehlungen, DGI Benchmark, PDF |
| Governance Audit | €149 | + 12-Wochen Trends, Peer Matrix, Executive Summary, PDF |

---

## Technische Umsetzung

### Was sich ändert

#### 1. DAO Matrix Page (`/matrix`)

**Entfernen:**
- "X more DAOs hidden" Banner + Lock-Overlay
- "Sign in to unlock" Button am Tabellenende
- Row-Blurring für nicht-angezeigte DAOs
- Login-Gate für Sortierung

**Beibehalten:**
- CSV/JSON Export Buttons — aber mit Subscription-Gate
- Bei Klick auf CSV/JSON ohne Subscription → Modal: "Subscribe for full export access — €19/mo"

**Hinzufügen:**
- Subtle CTA unterhalb der Tabelle: "Need deeper analysis? Get a Governance Audit →" (Link zu /pricing)
- "Export as CSV/JSON" Buttons sichtbar aber mit 🔒 Icon wenn nicht subscribed

#### 2. DAO Detail Pages (`/matrix/{dao-slug}`)

**Entfernen:**
- "Sign in to unlock more charts" Overlay auf HPR, DEI, PDI, GPI Charts
- "Subscribe for full access to all charts — €19/mo" Buttons
- Blur-Overlay auf gesperrten Charts

**Beibehalten:**
- Gesamtes Layout und Design (es sieht schon gut aus)
- Alle bestehenden Charts und Daten

**Hinzufügen:**
- CTA-Banner am Seitenende:

```
┌──────────────────────────────────────────────────────────────┐
│  📊 Want actionable insights, not just data?                  │
│                                                              │
│  Our AI-powered reports analyze what these numbers mean for   │
│  your DAO — with specific recommendations and benchmarks.     │
│                                                              │
│  [Get Free Check]  [Deep Dive — €49]  [Full Audit — €149]   │
└──────────────────────────────────────────────────────────────┘
```

- Optional: Mini-Teaser für Report-Inhalt (z.B. "Your HPR is 9.0 (A+) — what does that mean? Our Deep Dive explains.")

#### 3. Paywall-Logik Cleanup

```typescript
// ALT: Komplexe Gating-Logik
const visibleDAOs = isSubscribed ? allDAOs : allDAOs.slice(0, 5);
const showChart = (chart) => chart === 'GVS' || isLoggedIn || isSubscribed;

// NEU: Einfach
const visibleDAOs = allDAOs; // Alle sichtbar
const showChart = () => true; // Alle Charts offen
const canExport = isSubscribed; // Export bleibt gated
```

#### 4. Auth-Flow Anpassung

- "Sign in" bleibt im Header für Account-Management
- Aber kein "Sign in to unlock" mehr auf Content-Seiten
- Login ist nur nötig für: Export, Report-Kauf, Account-Dashboard

---

## Neue Analytics Events

### Bestehende Events die bleiben

Alle bestehenden Events bleiben unverändert: `cta_view`, `cta_click`, `section_view`, `scroll_25/50/75`, `time_on_page`, `engaged_5s/15s/30s/60s`, `rankings_view`, `guide_view`, `row_expand`, `quick_check_start`, `quick_check_complete`, `check_page_view`, `checkout_complete`, `onboarding_banner_shown`, `onboarding_banner_dismissed`

### Neue Events

```typescript
// --- DAO Matrix Events ---

// User klickt auf eine DAO-Zeile in der Matrix (→ Detail Page)
track("matrix_dao_click", {
  dao: "radiant-capital",    // DAO slug
  rank: 1,                   // Position in der Tabelle
  source: "matrix"           // Unterscheidung zu Rankings-Klick
});

// User sortiert die Matrix
track("matrix_sort", {
  column: "HPR",             // GVS, HPR, DEI, PDI, GPI, 7d
  direction: "desc"          // asc | desc
});

// User filtert nach Kategorie
track("matrix_filter", {
  category: "DeFi"           // All, DeFi, Infrastructure, Public Goods
});

// User nutzt die Suche
track("matrix_search", {
  query: "lido",             // Suchbegriff (lowercase)
  results: 1                 // Anzahl Ergebnisse
});

// User klickt Export aber hat keine Subscription
track("export_gate_hit", {
  format: "csv",             // csv | json
  dao_count: 46              // Wie viele DAOs in der aktuellen Ansicht
});

// --- DAO Detail Page Events ---

// User landet auf einer DAO Detail-Seite
track("dao_detail_view", {
  dao: "radiant-capital",
  dgi_score: 8.6,
  category: "DeFi",
  rank: 1
});

// User interagiert mit einem spezifischen Chart
track("dao_chart_interact", {
  dao: "radiant-capital",
  chart: "HPR",              // GVS, HPR, DEI, PDI, GPI
  action: "hover"            // hover | zoom | click
});

// User scrollt zum Report-CTA am Seitenende
track("dao_detail_report_cta_view", {
  dao: "radiant-capital"
});

// User klickt Report-CTA
track("dao_detail_report_cta_click", {
  dao: "radiant-capital",
  tier: "deep_dive"          // free_check | deep_dive | governance_audit
});

// --- Share Events (für zukünftiges Social Sharing) ---

// User kopiert die URL einer DAO Detail-Seite
// (optional, nur wenn Share-Button implementiert wird)
track("dao_share", {
  dao: "radiant-capital",
  method: "copy_link"        // copy_link | twitter | discord
});
```

### Event-Übersicht Tabelle

| Event | Trigger | Key Properties |
|-------|---------|----------------|
| `matrix_dao_click` | Klick auf DAO-Zeile in Matrix | dao, rank |
| `matrix_sort` | Spalte sortieren | column, direction |
| `matrix_filter` | Kategorie-Filter wählen | category |
| `matrix_search` | Suche in Matrix | query, results |
| `export_gate_hit` | Export-Klick ohne Subscription | format |
| `dao_detail_view` | DAO Detail-Seite laden | dao, dgi_score, rank |
| `dao_chart_interact` | Chart Hover/Click | dao, chart |
| `dao_detail_report_cta_view` | Report-CTA scrollt in View | dao |
| `dao_detail_report_cta_click` | Report-CTA Klick | dao, tier |
| `dao_share` | Share-Button Klick (optional) | dao, method |

### Funnel-Update

Der neue Haupt-Funnel wird:

```
matrix_dao_click → dao_detail_view → dao_chart_interact → dao_detail_report_cta_view → dao_detail_report_cta_click → quick_check_start → quick_check_complete → checkout_complete
```

---

## SEO Bonus

Offene DAO-Seiten werden indexierbar. Jede der 46 DAO-Detail-Seiten wird eine eigene Landing Page:

```
chainsights.one/matrix/radiant-capital → "Radiant Capital Governance Score | ChainSights"
chainsights.one/matrix/ens → "ENS Governance Score | ChainSights"
chainsights.one/matrix/lido → "Lido Governance Score | ChainSights"
```

Empfehlung: OG Meta Tags pro DAO-Seite für Social Sharing:

```html
<meta property="og:title" content="Radiant Capital — DGI Score 8.6 (A) | ChainSights" />
<meta property="og:description" content="Radiant Capital ranks #1 in the Decentralized Governance Index. Human Participation: 10.0 (A+), Delegate Engagement: 10.0 (A+)." />
<meta property="og:image" content="https://chainsights.one/og/radiant-capital.png" />
```

Das bedeutet: Wenn jemand den Link in Discord/X/LinkedIn teilt, sieht man sofort Score + Rang. Maximale Shareability.

---

## Migration / Rollout

1. Feature-Flag: `OPEN_MATRIX=true` — ermöglicht schnelles Rollback
2. Paywall-Logik ausschalten (Matrix + Detail Pages)
3. Report-CTAs auf Detail-Seiten hinzufügen
4. Export-Gate beibehalten (nur UI-Anpassung: 🔒 Icon statt versteckte Buttons)
5. Neue Events implementieren
6. OG Meta Tags pro DAO-Seite (kann als separater PR)
7. Monitoring: Vergleich Traffic/Engagement vorher vs. nachher

---

## Acceptance Criteria

- [ ] DAO Matrix zeigt alle 46 DAOs ohne Login
- [ ] Sortierung und Filter funktionieren ohne Login
- [ ] DAO Detail-Seiten zeigen alle 5 Charts ohne Login
- [ ] CSV/JSON Export bleibt hinter Subscription-Gate
- [ ] Report-CTA Banner auf jeder DAO Detail-Seite
- [ ] "Sign in to unlock" nirgends mehr auf Content-Seiten
- [ ] Alle neuen Events feuern korrekt
- [ ] `matrix_dao_click` enthält DAO-Name und Rang
- [ ] `dao_detail_view` enthält Score und Kategorie
- [ ] `export_gate_hit` trackt blocked Export-Versuche
- [ ] Feature-Flag für schnelles Rollback
- [ ] OG Meta Tags pro DAO-Seite (optional, kann Follow-up sein)
- [ ] Bestehende Events bleiben unverändert

---

## Files to Create/Touch

1. Matrix Page — Paywall-Logik entfernen
2. DAO Detail Page — Chart-Locks entfernen
3. DAO Detail Page — Report CTA Banner Component
4. Export Buttons — Subscription-Gate UI
5. Analytics Events — neue Tracking-Calls
6. OG Meta Tags — dynamisch pro DAO
7. Feature Flag config

---

## Out of Scope

- Comparison Mode (bleibt Subscription Feature, separates Ticket)
- Share Buttons auf Detail-Seiten (Future Enhancement)
- OG Image Generation pro DAO (kann als separater Task)
- Änderungen an Pricing/Tiers
- Report-Generierung oder -Inhalte
