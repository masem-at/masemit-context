# 🔧 FIX-SPEC: SPA Route-Change Detection in tracker.js

**Priorität:** CRITICAL – Blocker, heute müssen Launch-Posts raus
**Zugewiesen an:** masemIT Analytics Team
**Erstellt:** 2026-02-03
**Abhängig von:** BUG-analytics-tracker-coverage
**Input von:** ChainSights Team Analyse (Winston, Murat)
**Status:** ✅ BUG-1 GEFIXT (2026-02-03)

---

## ✅ Resolution (2026-02-03)

### Tatsächlicher Root Cause

Die initiale Diagnose war **falsch**. tracker.js hatte die SPA-Hooks bereits korrekt implementiert (Zeilen 272-296 in `public/tracker.js`):

```javascript
// Bereits vorhanden in tracker.js:
hookHistoryMethod('pushState');
hookHistoryMethod('replaceState');
window.addEventListener('popstate', ...);
window.addEventListener('hashchange', ...);
```

**Das eigentliche Problem:** `strategy="beforeInteractive"` im Script-Tag lud tracker.js *bevor* Next.js seinen Router initialisierte. Next.js überschrieb danach `history.pushState` mit seiner eigenen Version → die Tracker-Hooks wurden nie aufgerufen.

### Tatsächlicher Fix

**Ein Wort ändern** in den Projekt-Layouts:

```diff
- <Script src="..." strategy="beforeInteractive" />
+ <Script src="..." strategy="afterInteractive" />
```

**Betroffene Dateien (pro Projekt):**
- ChainSights: `src/app/layout.tsx`
- tellingCube: `src/app/layout.tsx`
- hoki.help: `app/layout.tsx`
- masem.at: `app/layout.tsx`

### Verifizierung

DevTools → Network → Filter `analytics.masem.at`:
- Navigation `/` → `/governance-index` → `/pricing` → `/matrix`
- ✅ Für jeden Klick erscheint ein POST-Request mit korrektem `path`

---

## Ursprüngliche Diagnose (überholt)

~~| Check | Status | Ergebnis |~~
~~|---|---|---|~~
~~| SPA Route-Change | 🔴 FEHLT | Kein pageView bei Client-Side Navigation |~~

**Korrektur:**

| Check | Status | Ergebnis |
|---|---|---|
| Root Layout Einbindung | ✅ OK | tracker.js ist in `src/app/layout.tsx` |
| Script-Attribute | ✅ OK | `data-project`, Engagement-Buckets korrekt |
| CSP Headers | ✅ OK | `analytics.masem.at` in `script-src` und `connect-src` erlaubt |
| SPA Route-Change | ✅ OK | History-Hooks waren bereits implementiert (Z. 272-296) |
| `beforeInteractive` Timing | 🔴 **ROOT CAUSE** | Script lief vor Next.js Router → Hooks überschrieben |
| Safari/iOS | 🟡 Offen | Separates Issue |

---

## Offene Items (nicht mehr in diesem Scope)

### Bot-Detection (BUG-2)

Weiterhin offen - tracker.js hat keine Bot-Filterung. Vorgeschlagener Code:

```javascript
function isBot() {
  if (navigator.webdriver) return true;
  var ua = navigator.userAgent || '';
  var botPatterns = /bot|crawl|spider|slurp|facebookexternalhit|Lighthouse|PageSpeed|HeadlessChrome/i;
  if (botPatterns.test(ua)) return true;
  if (window.location.hostname.includes('.vercel.app')) return true;
  return false;
}
```

### Safari/iOS (BUG-3)

Weiterhin offen - benötigt Browser-Test.

---

## Akzeptanzkriterien

- [x] pageView wird bei JEDER Navigation gesendet (initial + SPA route change)
- [x] Kein Doppel-Tracking bei gleichem Pfad (Debounce bereits in tracker.js)
- [ ] Bot-Traffic (navigator.webdriver, bekannte Bot-UAs) wird gefiltert → **Offen**
- [ ] Vercel Preview-Deployments werden nicht getrackt → **Offen**
- [x] tracker.js funktioniert mit `afterInteractive`
- [x] Kein Breaking Change in der API für bestehende Projekte

## Validierungstest

✅ **Durchgeführt (2026-02-03):**

```
Browser-Test:
1. DevTools → Network → Filter "analytics.masem.at"
2. Navigieren: / → /governance-index → /pricing → /matrix
3. ✅ Für JEDEN Klick erscheint ein POST-Request
4. ✅ Payload enthält korrekten path
```

## Lessons Learned

1. **Initiale Diagnose war falsch:** Wir nahmen an, der Tracker hätte keine SPA-Hooks. Code-Review zeigte: Hooks waren da.
2. **Script-Timing in Next.js:** `beforeInteractive` lädt Scripts vor Framework-Initialisierung. Wenn das Script Browser-APIs wrapped (wie `history.pushState`), kann das Framework diese Wrapper überschreiben.
3. **Empfehlung für alle Projekte:** Analytics-Tracker sollten `afterInteractive` verwenden, nicht `beforeInteractive`.

## Timeline

- **2026-02-03 vormittag:** Bug-Report erstellt, falsche Root-Cause-Analyse
- **2026-02-03 Party Mode:** Code-Review in tracker.js → Hooks gefunden → Timing-Problem identifiziert
- **2026-02-03:** Fix deployed (`afterInteractive`), verifiziert
