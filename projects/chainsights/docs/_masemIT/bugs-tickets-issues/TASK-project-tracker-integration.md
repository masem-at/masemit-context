# 🔧 TASK: Tracker-Integration in Projekten verifizieren & fixen

**Priorität:** CRITICAL – heute Launch-Posts ChainSights, morgen tellingCube  
**Zugewiesen an:** ChainSights + tellingCube Projekt-Teams  
**Erstellt:** 2026-02-03  
**Abhängigkeit:** Parallel zu BUG-analytics-tracker-coverage (Analytics Team)  

---

## Warum jetzt

Analyse zeigt: Der Tracker erfasst nur ~8% des tatsächlichen Traffics. Heute und morgen gehen Launch-Posts raus. Ohne funktionierendes Tracking verpassen wir die wichtigsten Conversion-Daten.

## Aufgaben pro Projekt

### ChainSights (chainsights.one) – HEUTE vor Post

**1. Tracker-Einbindung prüfen**
```
Frage: Wo ist tracker.js eingebunden?
Soll:   app/layout.tsx (Root Layout → gilt für ALLE Seiten)
Fehler: Nur in app/page.tsx oder app/(home)/page.tsx
```
- [ ] `<Script>` Tag in root `layout.tsx` verifizieren
- [ ] Falls nicht vorhanden: tracker.js in root layout einbinden

**2. Route-Change-Tracking prüfen**
```
Frage: Wird bei SPA-Navigation ein neuer pageView gesendet?
Test:   Browser DevTools → Network Tab → von / nach /pricing navigieren
Soll:   Neuer POST an Tracking-API sichtbar
Fehler: Kein Request bei Navigation
```
- [ ] Testen: Klick von Homepage auf jede Hauptseite → prüfen ob Event in Network Tab
- [ ] Falls nicht: `usePathname()` Hook implementieren der bei Änderung pageView sendet

**3. Schnelltest Safari/iOS**
- [ ] chainsights.one auf iPhone öffnen (oder Safari Responsive Mode)
- [ ] Console auf Fehler prüfen (besonders `tracker.js` Ladefehler)
- [ ] Network Tab: wird der Track-Request gesendet?

**4. Bot-Filter prüfen**
- [ ] Wird `navigator.webdriver` gecheckt bevor Events gesendet werden?
- [ ] Werden Preview-Deployments (`*.vercel.app`) von Tracking ausgenommen?

**5. Validierung nach Fix**
```bash
# 30 Minuten nach Fix: Neon-Events prüfen
SELECT path, COUNT(*) 
FROM events 
WHERE project_id = 'chainsights' 
  AND timestamp > NOW() - INTERVAL '30 minutes'
GROUP BY path
ORDER BY count DESC;
```
- [ ] Ergebnis zeigt Events von `/`, `/governance-index`, `/pricing`, `/matrix` etc.

---

### tellingCube (tellingcube.com) – VOR MORGEN

Gleiche Checkliste wie oben, zusätzlich:

- [ ] Tracker überhaupt eingebunden? (Projekt evtl. noch ohne Analytics-Integration)
- [ ] Wenn nicht: tracker.js einbinden (root layout + route-change-detection)
- [ ] Alle relevanten Custom Events definiert? (pageView, cta_click, signup_start, etc.)
- [ ] Ein manueller Durchlauf des geplanten Launch-Funnels:
  - Landing Page → Feature-Seite → Pricing → Signup
  - Jeder Step muss in Neon events auftauchen

---

## Minimaler Fix wenn Zeit knapp

Falls kein Full-Fix möglich vor den Posts, mindestens:

1. **tracker.js in root layout.tsx** (5 Minuten) → deckt alle Seiten ab bei Hard-Navigation
2. **Route-Change-Hook** (15 Minuten) → deckt SPA-Navigation ab

Das alleine hebt die Abdeckung von ~8% auf geschätzt 60-70%. Bot-Filter und Safari-Fix können danach kommen.

## Rohdaten

Detaillierte Analyse siehe: `BUG-analytics-tracker-coverage.md` (Analytics Team Ticket)
