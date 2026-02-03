# 🚨 BUG: Tracker erfasst nur ~8% der tatsächlichen Page Loads

**Priorität:** CRITICAL – Blocker für heute/morgen geplante Launches  
**Zugewiesen an:** masemIT Analytics Team  
**Erstellt:** 2026-02-03  
**Betrifft:** tracker.js / Event-Ingestion-Pipeline  
**Status:** ✅ ROOT CAUSE GEFUNDEN & BUG-1 GEFIXT (2026-02-03)

---

## ✅ Resolution (2026-02-03)

**Root Cause:** `strategy="beforeInteractive"` in ChainSights `layout.tsx`. tracker.js hat die SPA-Hooks (pushState/replaceState/popstate) bereits korrekt implementiert (Zeilen 272-296). Aber `beforeInteractive` lud das Script VOR der Next.js Router-Initialisierung → Next.js überschrieb danach `history.pushState` mit seiner eigenen Version → unsere Hooks waren weg.

**Fix:** Ein Wort – `strategy="afterInteractive"` in `src/app/layout.tsx`. Deployed & verifiziert via DevTools (pageView-Ping mit `path: "/matrix"` nach SPA-Navigation bestätigt).

**Offen:** BUG-2 (Bot-Detection) und BUG-3 (Safari/iOS) – siehe unten.

---

## Kontext

Heute und morgen gehen Launch-Posts raus (ChainSights neues Feature + tellingCube). Wenn der Tracker nicht funktioniert, haben wir keinen Funnel, keine Conversion-Daten, keine Ahnung was die Launches bringen. Jeder verlorene Tag ist unwiederbringlich.

## Befund

Vergleich von Vercel-Logs (728 Rows, ~12h Zeitfenster 2026-02-02 17:45 – 2026-02-03 05:41) mit Neon events-Tabelle (85 Rows, selber Zeitraum) für das Projekt **ChainSights**:

| Metrik | Vercel (Quelle der Wahrheit) | Neon (unser Tracker) |
|---|---|---|
| Echte Human-Page-Requests | 211 | 17 pageViews |
| Tracking-Abdeckung gesamt | 100% | **~8%** |
| Abgedeckte Pfade | 50+ verschiedene | **nur `/`** |
| Tracking-Abdeckung auf `/` | 25 Visits | 17 pageViews (~68%) |

## Drei Probleme

### ✅ BUG-1: Tracker feuert nur auf Homepage `/` — GEFIXT

**Schweregrad:** Critical  
**Status:** ✅ Gefixt (2026-02-03)  
**Root Cause:** `strategy="beforeInteractive"` → Next.js überschrieb History-Hooks nach Tracker-Init  
**Fix:** `strategy="afterInteractive"` in `src/app/layout.tsx`  
**Verifiziert:** DevTools zeigt `pageView` Pings mit korrekten Pfaden nach SPA-Navigation

### BUG-2: Bot-Traffic wird nicht gefiltert

**Schweregrad:** High  
**Symptom:** 64 von 85 Neon-Events kommen von `Linux/Chrome/desktop` mit wechselnden device_ids, teilweise in Paaren im Sekundentakt. Nur 22 Events sind `Windows/Chrome` (wahrscheinlich echter Traffic).  
**Vermutete Ursache:** Vercel Speed Insights, Lighthouse-Checks, Monitoring oder Headless-Browser-Bots werden als echte User getrackt.  
**Impact:** Event-Zahlen sind aufgeblasen, Unique-Visitor-Counts und Conversion-Rates verfälscht.

**Fix prüfen:**
- [ ] Bot-Detection in tracker.js implementiert? (`navigator.webdriver`, bekannte Bot-UAs)
- [ ] Serverseitiger Bot-Filter auf der Ingestion-API?
- [ ] Vercel Speed Insights / Preview-Deployments erzeugen Events?

### BUG-3: Mobile/Safari wird nicht getrackt

**Schweregrad:** High  
**Symptom:** Vercel zeigt 7+ iPhone-Visits auf `/`, Neon hat 0 Events von iOS/Safari.  
**Vermutete Ursache:** Safari ITP blockt Third-Party-Tracking, oder tracker.js hat einen Safari-spezifischen Ladefehler, oder AdBlocker-Rate auf Mobile ist höher.  
**Impact:** Gesamter Mobile-Traffic ist unsichtbar. Bei Launch-Posts über Social Media (viele Mobile-User) besonders kritisch.

**Fix prüfen:**
- [ ] Console-Errors in Safari/iOS prüfen
- [ ] Wird tracker.js als First-Party-Script geladen (same domain)?
- [ ] Werden Events an eine First-Party-API gesendet oder an eine externe Domain?

## Akzeptanzkriterien für Fix

- [x] pageView Events werden auf ALLEN Seiten gefeuert, nicht nur `/`
- [x] SPA-Navigation tracked korrekt (Route-Change erzeugt neuen pageView)
- [ ] Bot-Traffic gefiltert (kein `navigator.webdriver`, keine bekannten Bot-UAs)
- [ ] Safari/iOS erzeugt Events (Console-Error-frei)
- [ ] Vergleichstest: Vercel-Logs vs Neon-Events für 1h zeigt >80% Abdeckung

## Rohdaten

Analyse basiert auf:
- `vercel-logs_result.csv` (728 Rows, nur chainsights.one)
- `neon-events-results.csv` (85 Rows, nur chainsights)
- Zeitfenster: 2026-02-02 17:45 UTC – 2026-02-03 05:41 UTC
