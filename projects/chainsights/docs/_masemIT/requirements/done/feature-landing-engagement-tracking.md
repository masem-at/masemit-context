# Feature: Landing Page Engagement Tracking

**Project:** masemIT Analytics → ChainSights Integration
**Priority:** High — 99% der User sehen nur die Landing Page und gehen. Wir wissen nicht warum.

---

## Problem

93 Page Views, 81 davon Landing Page, 3 gehen weiter zu /rankings. Was passieren die anderen 78? Möglichkeiten:

1. Sie sehen die Headline und gehen sofort (< 5 Sekunden)
2. Sie scrollen ein Stück, finden nichts Relevantes, gehen
3. Sie scrollen bis zum CTA, der CTA überzeugt nicht
4. Sie scrollen komplett durch, sind interessiert, aber der nächste Schritt ist unklar
5. Sie klicken auf den CTA, aber das Click-Event feuert nicht (Tracking-Bug)

Ohne Daten raten wir. Mit diesen Events wissen wir es.

---

## Events

### 1. Scroll Depth (Intersection Observer)

Unsichtbare Marker-Elemente an definierten Positionen der Landing Page. Feuern **einmal pro Session** wenn der Viewport sie erreicht.

| Event | Trigger | Properties |
|-------|---------|------------|
| `scroll_25` | User scrollt 25% der Seite | `page` |
| `scroll_50` | User scrollt 50% der Seite | `page` |
| `scroll_75` | User scrollt 75% der Seite | `page` |
| `scroll_100` | User erreicht Footer | `page` |

**Implementierung:**

```tsx
// Unsichtbare Marker in der Landing Page
<div data-scroll-marker="25" />  // nach Hero Section
<div data-scroll-marker="50" />  // nach Features/Benefits
<div data-scroll-marker="75" />  // nach Social Proof / Rankings Teaser
<div data-scroll-marker="100" /> // vor Footer

// Generischer Observer (in der Analytics-Komponente)
useEffect(() => {
  const markers = document.querySelectorAll('[data-scroll-marker]')
  const fired = new Set<string>()

  const observer = new IntersectionObserver(
    (entries) => {
      entries.forEach((entry) => {
        if (entry.isIntersecting) {
          const depth = entry.target.getAttribute('data-scroll-marker')
          if (depth && !fired.has(depth)) {
            fired.add(depth)
            window.trackEvent?.(`scroll_${depth}`, { page: window.location.pathname })
            observer.unobserve(entry.target)
          }
        })
      })
    },
    { threshold: 0.1 }
  )

  markers.forEach((m) => observer.observe(m))
  return () => observer.disconnect()
}, [])
```

### 2. Time on Page (Engagement Timer)

Feuert in Stufen — zeigt ob jemand wirklich liest oder sofort abspringt.

| Event | Trigger | Properties |
|-------|---------|------------|
| `engaged_5s` | User ist 5 Sekunden auf der Seite | `page` |
| `engaged_15s` | User ist 15 Sekunden auf der Seite | `page` |
| `engaged_30s` | User ist 30 Sekunden auf der Seite | `page` |
| `engaged_60s` | User ist 60 Sekunden auf der Seite | `page` |

**Implementierung:**

```tsx
useEffect(() => {
  const milestones = [5, 15, 30, 60]
  const timers: NodeJS.Timeout[] = []

  milestones.forEach((seconds) => {
    timers.push(
      setTimeout(() => {
        window.trackEvent?.(`engaged_${seconds}s`, { page: window.location.pathname })
      }, seconds * 1000)
    )
  })

  // Pausieren wenn Tab nicht sichtbar
  const handleVisibility = () => {
    if (document.hidden) {
      timers.forEach(clearTimeout)
    }
  }
  document.addEventListener('visibilitychange', handleVisibility)

  return () => {
    timers.forEach(clearTimeout)
    document.removeEventListener('visibilitychange', handleVisibility)
  }
}, [])
```

**Wichtig:** Timer pausieren bei `document.hidden` — sonst zählt ein vergessener Tab als "engaged".

### 3. Section Visibility (Was wird gelesen?)

Statt nur Prozent-Marker: Welche **inhaltlichen Sektionen** werden gesehen?

| Event | Trigger | Properties |
|-------|---------|------------|
| `section_view` | Sektion wird 50% sichtbar für > 1 Sekunde | `section`, `page` |

Sektionen der Landing Page (Beispiel, an tatsächliche IDs anpassen):

| Section ID | Inhalt |
|------------|--------|
| `hero` | Headline + Primary CTA |
| `value_prop` | "Wallets lie. We don't." Erklärung |
| `how_it_works` | GVS / Scoring Erklärung |
| `rankings_preview` | Top DAOs Teaser |
| `pricing` | Report Tiers |
| `cta_bottom` | Bottom CTA |

```tsx
// Pro Sektion:
<section data-track-section="hero">
  ...
</section>

// Observer mit 1-Sekunden Delay
const observer = new IntersectionObserver(
  (entries) => {
    entries.forEach((entry) => {
      const section = entry.target.getAttribute('data-track-section')
      if (entry.isIntersecting) {
        // Starte 1s Timer
        entry.target._timer = setTimeout(() => {
          window.trackEvent?.('section_view', {
            section,
            page: window.location.pathname,
          })
          observer.unobserve(entry.target)
        }, 1000)
      } else {
        // Abbrechen wenn zu schnell vorbei gescrollt
        clearTimeout(entry.target._timer)
      }
    })
  },
  { threshold: 0.5 }
)
```

---

## Dashboard: Landing Page Engagement View

### Scroll Funnel (neue Ansicht in analytics.masem.at)

```
Landing Page Scroll Funnel (letzte 7 Tage)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Page Load     ███████████████████████████  81  (100%)
Scroll 25%    ████████████████████         62  (77%)
Scroll 50%    ██████████████               45  (56%)
Scroll 75%    ████████                     25  (31%)
Scroll 100%   ███                          12  (15%)
CTA Click     █                             3  (4%)
```

→ "Aha! 44% der Leute kommen nicht über die Hälfte der Seite. Problem ist above-the-fold Content."

ODER:

```
Page Load     ███████████████████████████  81  (100%)
Scroll 25%    █████████████████████████    78  (96%)
Scroll 50%    ████████████████████████     75  (93%)
Scroll 75%    ██████████████████████       70  (86%)
Scroll 100%   ████████████████████         65  (80%)
CTA Click     █                             3  (4%)
```

→ "Leute scrollen durch die GANZE Seite und klicken trotzdem nicht. Problem ist der CTA selbst (Text, Farbe, Angebot)."

### Engagement Breakdown

```
Time on Page Distribution
━━━━━━━━━━━━━━━━━━━━━━━━

< 5s      ████████████████  42  (52%)  ← Instant Bounce
5-15s     ████████           20  (25%)  ← Scanned, left
15-30s    ████               10  (12%)  ← Read something
30-60s    ██                  6  (7%)   ← Interested
> 60s     █                   3  (4%)   ← Engaged
```

### Section Heatmap

```
Section Views (of 81 landing visitors)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

hero              81  ████████████████████  100%
value_prop        68  ████████████████      84%
how_it_works      52  █████████████         64%
rankings_preview  41  ██████████            51%
pricing           22  ██████                27%  ← Drop-off!
cta_bottom        18  █████                 22%
```

→ "Nur 27% sehen überhaupt die Pricing Section. Entweder Seite ist zu lang oder how_it_works verliert sie."

---

## Implementierungs-Aufwand

| Teil | Aufwand | Wo |
|------|---------|-----|
| Scroll Marker + Observer | 1h | Analytics .js Komponente |
| Time Engagement Timer | 1h | Analytics .js Komponente |
| Section Tracking | 1h | ChainSights Landing Page (data-attributes) |
| Dashboard Scroll Funnel | 2h | analytics.masem.at |
| Dashboard Engagement View | 2h | analytics.masem.at |
| **Total** | **~7h** | |

---

## Nicht in Scope

- Mouse Movement / Heatmaps (zu invasiv, zu viel Daten)
- Rage Click Detection (nice-to-have, später)
- Video/Animation View Tracking (kein Video auf Landing)
- A/B Testing Framework (kommt separat wenn nötig)

---

## Erwarteter Outcome

Nach 1 Woche mit diesen Events können wir eine von drei Diagnosen stellen:

**Diagnose A: "Sie lesen nicht"**
→ 50%+ bouncen in < 5s, Scroll < 25%
→ Aktion: Headline/Hero komplett überarbeiten

**Diagnose B: "Sie lesen, aber der CTA überzeugt nicht"**
→ 70%+ scrollen bis 75%, aber < 5% klicken CTA
→ Aktion: CTA Text, Farbe, Angebot ändern

**Diagnose C: "Sie klicken, aber der Flow danach ist kaputt"**
→ 10%+ CTA Klicks, aber 0 Conversions
→ Aktion: /check Flow Page debuggen (→ bereits im Refactor)

Jede Diagnose hat eine klare nächste Aktion. Kein Raten mehr.
