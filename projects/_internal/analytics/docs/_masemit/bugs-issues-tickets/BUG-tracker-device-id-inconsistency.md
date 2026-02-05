# BUG: Tracker vergibt inkonsistente device_ids bei SPA-Navigation

**Projekt:** masemIT Analytics / ChainSights Tracker
**Erstellt:** 2026-02-05
**Gefixt:** 2026-02-05
**Priorität:** 🔴 CRITICAL
**Status:** ✅ GEFIXT
**Betrifft:** tracker.js (device_id Generierung/Persistierung)
**Fix-Commit:** b56ec33 (tracker v1.4.0)

---

## Problem

Der Analytics-Tracker vergibt bei manchen SPA-Navigationen (Client-Side Routing) **neue device_ids** statt die bestehende ID beizubehalten. Das führt dazu, dass Events desselben Besuchers nicht verknüpfbar sind.

**Auswirkung:** Funnels, Session-Analyse und User Journeys sind unbrauchbar, weil zusammengehörige Events verschiedenen "Besuchern" zugeordnet werden.

---

## Beweis aus der Datenbank (2026-02-05)

### Fall 1: US-Besucher – device_id wechselt bei Navigation

```
device_id: 431242508937552947 → /governance-index  um 09:00:25
device_id: 564605412860400734 → /matrix/balancer   um 09:01:09
device_id: 564605412860400734 → /matrix/gitcoin    um 09:01:09
device_id: 564605412860400734 → /matrix/safe       um 09:01:09
```

**Derselbe Besucher** navigiert von `/governance-index` zu `/matrix/*` – bekommt aber eine neue device_id. Der Funnel sieht zwei verschiedene Besucher: einer der nur `/governance-index` anschaut, und einer der nur `/matrix/*` anschaut.

### Fall 2: AL-Besucher – device_id bleibt konsistent ✅

```
device_id: 778799811813160107 → /governance-index   um 09:03:25
device_id: 778799811813160107 → cta_click            um 09:03:30
device_id: 778799811813160107 → /check               um 09:03:30
device_id: 778799811813160107 → check_page_view      um 09:03:31
```

Hier funktioniert alles korrekt – alle Events haben dieselbe device_id.

### Fall 3: Längere Session – device_id bleibt konsistent ✅

```
device_id: 988963527850210446 → /           um 16:05:16
device_id: 988963527850210446 → cta_click   um 16:05:23
device_id: 988963527850210446 → /rankings   um 16:05:26
device_id: 988963527850210446 → /           um 18:29:37  (2h später!)
device_id: 988963527850210446 → /rankings   um 18:30:18
```

Sogar über 2 Stunden hinweg bleibt die ID stabil.

---

## Analyse

Der Bug tritt **nicht immer** auf – manche Sessions haben konsistente IDs (Fall 2 & 3), andere nicht (Fall 1). Mögliche Ursachen:

### Hypothese A: Cookie/localStorage wird nicht gesetzt bevor Navigation feuert
Bei der ersten Seitenladung wird die device_id generiert und gespeichert. Wenn der Besucher sehr schnell navigiert (innerhalb <1s), könnte der zweite pageView feuern bevor der Cookie/localStorage-Write abgeschlossen ist → neue ID wird generiert.

**Passt zu Fall 1:** Navigation von `/governance-index` (09:00:25) zu `/matrix/balancer` (09:01:09) ist ~44 Sekunden – eigentlich lang genug. Aber die drei `/matrix/*` Events kommen alle um 09:01:09 – das sieht nach einem schnellen Tab-/Link-Wechsel aus, möglicherweise ein Pre-Render oder Multi-Tab Szenario.

### Hypothese B: Verschiedene Browser-Tabs / Pre-Rendering
Wenn der Besucher Links in neuen Tabs öffnet (Cmd+Click), bekommt jeder Tab möglicherweise eine eigene device_id, je nachdem wie der Tracker initialisiert wird.

**Passt zu Fall 1:** Drei `/matrix/*` Seiten innerhalb von <1 Sekunde → der Besucher hat wahrscheinlich mehrere Links gleichzeitig geöffnet.

### Hypothese C: Tracker-Initialisierung bei Route Change
Der `afterInteractive` Fix (BUG-analytics-tracker-coverage.md) hat die Ladereihenfolge geändert. Möglicherweise wird bei bestimmten Route Changes der Tracker neu initialisiert und generiert dabei eine neue device_id statt die bestehende zu lesen.

---

## Was untersucht werden muss

1. **Wie wird die device_id persistiert?**
   - Cookie? localStorage? sessionStorage?
   - Wird sie synchron oder asynchron geschrieben?
   - Was passiert bei gleichzeitigen Schreibvorgängen (Race Condition)?

2. **Was passiert bei der Tracker-Initialisierung?**
   - Prüft der Tracker beim Start ob bereits eine device_id existiert?
   - Wird bei SPA Route Changes der Tracker neu initialisiert?
   - Gibt es einen Unterschied zwischen Hard Navigation und Client-Side Routing?

3. **Multi-Tab Verhalten**
   - Was passiert wenn ein Besucher mehrere Links gleichzeitig öffnet?
   - Wird die device_id pro Tab oder pro Browser-Session generiert?

4. **Reproduzierbarkeit**
   - Auf chainsights.one: `/governance-index` aufrufen, dann auf einen Matrix-Link klicken
   - Prüfen ob die Events in der DB dieselbe device_id haben
   - Auch testen: Rechtsklick → "In neuem Tab öffnen"

---

## Erwartetes Verhalten

- **Ein Besucher = Eine device_id** für die gesamte Browser-Session
- device_id bleibt stabil über SPA-Navigation, Tab-Wechsel und Page Reloads
- device_id wird beim ersten Besuch generiert und zuverlässig persistiert
- Alle nachfolgenden Events (pageView, custom Events) verwenden dieselbe ID

---

## Sofort-Workaround

Keiner möglich. Ohne konsistente device_ids sind Funnels prinzipiell nicht auswertbar. Der Bug muss im Tracker behoben werden.

---

## Verknüpfte Issues

- BUG-analytics-tracker-coverage.md → ✅ GEFIXT (beforeInteractive → afterInteractive) – könnte verwandt sein
- TASK-funnel-editor-event-data-filter.md → Funnel-Config ist korrekt, Daten fehlen wegen diesem Bug
- Funnel "Main Conversion" auf ChainSights zeigt 0% Conversion ab Step 2 trotz nachweislicher User Journeys

---

## Akzeptanzkriterien

- [x] device_id bleibt bei SPA-Navigation (Next.js Client-Side Routing) stabil
- [x] device_id bleibt bei Multi-Tab Szenarien konsistent (selber Browser)
- [ ] Funnel "Main Conversion" zeigt korrekte Zahlen für Step 2+ nach dem Fix
- [ ] Regressions-Test: masem.at Funnels funktionieren weiterhin
- [ ] DB-Validierung: Query bestätigt dass Sessions mit mehreren Pfaden dieselbe device_id haben

---

## Resolution (2026-02-05)

**Root Cause:** Die deviceId wurde einmalig beim Script-Load in eine Variable gecached. Bei Multi-Tab Szenarien (Cmd+Click auf mehrere Links) starteten alle Tabs fast gleichzeitig und lasen localStorage bevor ein anderer Tab geschrieben hatte → jeder Tab generierte eine eigene ID.

**Fix in tracker.js v1.4.0:**

1. **Fresh Read bei jedem Event:** `buildPayload()` ruft jetzt `getDeviceId()` auf statt die gecachte Variable zu verwenden. So bekommt Tab 2 die ID von Tab 1, wenn Tab 1 zuerst geschrieben hat.

2. **Cross-Tab Sync:** Ein `storage` Event Listener aktualisiert den lokalen Cache wenn ein anderer Tab die deviceId ändert.

3. **Cached Fallback:** Bei Storage-Fehlern (Private Mode, Quota exceeded) wird eine konsistente ID innerhalb des Tabs verwendet statt bei jedem Event eine neue zu generieren.

**Commit:** b56ec33
