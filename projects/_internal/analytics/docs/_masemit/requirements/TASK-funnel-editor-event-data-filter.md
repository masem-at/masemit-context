# TASK: FunnelEditor – event_data Filter Support

**Projekt:** masemIT Analytics  
**Erstellt:** 2026-02-04  
**Priorität:** High  
**Status:** ⬜ OFFEN  
**Betrifft:** `FunnelEditor.tsx`, Funnel Query Engine

---

## Problem

Der FunnelEditor kann aktuell nur nach **event_name** und **path** filtern. Viele Events auf masem.at (und anderen Projekten) nutzen aber **event_data** (JSONB) als primäres Unterscheidungsmerkmal.

### Konkretes Beispiel: masem.at

Die Navigation trackt so:

| Aktion | event_name | event_data |
|--------|-----------|------------|
| Nav "Vision" | `nav_click` | `{ "target": "vision" }` |
| Nav "Products" | `nav_click` | `{ "target": "products" }` |
| Nav "Contact" | `nav_click` | `{ "target": "contact" }` |
| Social LinkedIn | `contact_click` | `{ "contact_type": "linkedin" }` |
| Produkt ChainSights | `project_click` | `{ "project_name": "chainsights" }` |

**Ohne event_data-Filter ist es unmöglich**, z.B. nur Klicks auf "Contact" von anderen nav_clicks zu unterscheiden. Ein Funnel-Step "User klickt auf Contact" lässt sich nicht konfigurieren.

### Entdeckter Fehler in bestehender Config

Der Funnel "Main Conversion" für masem.at war falsch konfiguriert:
- Step 1: `pageView` auf `/` → matcht nie (Seiten sind `/de`, `/en`, `/de/hoki` etc.)
- Step 2: `nav_click` mit Wert `de, en` → matcht nie (nav_clicks haben `target: "products"` etc., kein Feld `de`/`en`)

---

## Anforderung

### Muss (MVP)

1. **event_data Filter-Feld im FunnelEditor**
   - Pro Funnel-Step ein optionales Key-Value-Paar für event_data
   - UI: Textfeld "Event Data Key" + Textfeld "Event Data Value"
   - Beispiel: Key=`target`, Value=`contact`
   - Wird in der Funnel-Config als JSON gespeichert

2. **Query-Engine Erweiterung**
   - Funnel-Query muss `event_data->>'key' = 'value'` unterstützen
   - Wenn kein event_data-Filter gesetzt → bisheriges Verhalten (nur event_name + path)

3. **Path-Matching verbessern**
   - Mindestens: `starts_with` Support (z.B. `/de` matcht `/de`, `/de/hoki`, `/de/legal`)
   - Ideal: Dropdown mit Optionen `exact`, `starts_with`, `contains`, `regex`

### Muss (MVP) – Teil 2: Visual Examples Panel

4. **Collapsible Examples Panel im FunnelEditor**
   - Unterhalb der Steps ein aufklappbares "Examples"-Panel
   - Hilft Nutzern, die Konfiguration zu verstehen (Funnel-Setup ist selten)
   - Tabs zum Durchklicken verschiedener Szenarien

5. **Mindestens 3 Example-Tabs mit SVG-Visualisierungen**

   | Tab | Inhalt | Visualisierung |
   |-----|--------|----------------|
   | **PageView + Path** | Wie `exact` vs `starts_with` funktioniert | Path-Baum mit Matches (✓/✗) |
   | **Custom Event** | Wie `event_data` Filter matcht | event_name → event_data → Ergebnis |
   | **Multi-Step Flow** | Wie die device_id-Chain funktioniert | Step1 → Step2 → Step3 User-Journey |

6. **SVG-Trees als React-Komponenten**
   - Nicht statische Bilder, sondern dynamische Komponenten
   - Ermöglicht später Live-Preview mit aktuellen Werten
   - Konsistentes Styling mit dem Rest der App

### Soll (Nice-to-have)

7. **Event Data Autocomplete**
   - Basierend auf event_catalog: bekannte Keys und häufige Values vorschlagen
   - Reduziert Tippfehler und macht das UI intuitiver

8. **Mehrere event_data Filter pro Step**
   - AND-Verknüpfung: `target = "contact" AND contact_type = "form"`
   - Für komplexere Funnel-Definitionen

9. **Live-Preview im Side Panel**
   - Zeigt in Echtzeit wie die aktuelle Config matchen würde
   - Dynamische SVG-Trees mit den eingegebenen Werten

---

## Technische Details

### Aktuelle Event-Struktur (DB)

```
event_name: VARCHAR    ← z.B. "nav_click"
path:       VARCHAR    ← z.B. "/de"
event_data: JSONB      ← z.B. {"target": "contact"}
```

### Aktueller FunnelEditor (vereinfacht)

```
Step: [event_name ▼] [path____] [value____]
```

- `event_name`: Dropdown aus event_catalog
- `path`: Freitext, exaktes Matching
- `value`: unklar wogegen das matcht → **das ist das Problem**

### Vorgeschlagene Erweiterung

```
Step: [event_name ▼] [path____] [match: exact ▼] [data key____] [data value____]
```

### UI Mockup: Examples Panel

```
┌─────────────────────────────────────────────────────────┐
│  Edit Funnel: Main Conversion                      [×]  │
├─────────────────────────────────────────────────────────┤
│  Name: [Main Conversion________]  ☑ Default             │
│                                                         │
│  Steps:                                                 │
│  ┌─────────────────────────────────────────────────┐   │
│  │ 1. [Landing] [pageView ▼] [/de____] [starts_with▼]  │
│  │ 2. [Nav CTA] [nav_click▼] [_______] [target=contact]│
│  │ + Add Step                                          │
│  └─────────────────────────────────────────────────┘   │
│                                                         │
│  ▼ Examples – How to configure funnels                  │
│  ┌─────────────────────────────────────────────────┐   │
│  │ [PageView+Path] [Custom Event] [Multi-Step Flow]│   │
│  │ ─────────────────────────────────────────────── │   │
│  │                                                     │
│  │   ┌──────────┐      Matches:                       │
│  │   │ pageView │      ✓ /de                          │
│  │   └────┬─────┘      ✓ /de/hoki                     │
│  │        │            ✓ /de/legal                    │
│  │   path LIKE '/de%'  ✗ /en                          │
│  │   (starts_with)     ✗ /en/about                    │
│  │                                                     │
│  └─────────────────────────────────────────────────┘   │
│                                                         │
│  [Delete]                        [Cancel] [Save]        │
└─────────────────────────────────────────────────────────┘
```

### Example Tab 2: Custom Event

```
┌─────────────────────────────────────────────────────────┐
│ [PageView+Path] [Custom Event●] [Multi-Step Flow]       │
│ ─────────────────────────────────────────────────────── │
│                                                         │
│   ┌───────────┐                                         │
│   │ nav_click │  ← event_name                           │
│   └─────┬─────┘                                         │
│         │                                               │
│         ▼                                               │
│   ┌─────────────────┐                                   │
│   │ event_data      │                                   │
│   │ ->>'target'     │  ← filter key                     │
│   └─────┬───────────┘                                   │
│         │                                               │
│         ▼                                               │
│   = 'contact'          ← filter value                   │
│         │                                               │
│         ▼                                               │
│   ✓ {target:"contact"}    Matches                       │
│   ✗ {target:"vision"}     No match                      │
│   ✗ {target:"products"}   No match                      │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

### Example Tab 3: Multi-Step Flow

```
┌─────────────────────────────────────────────────────────┐
│ [PageView+Path] [Custom Event] [Multi-Step Flow●]       │
│ ─────────────────────────────────────────────────────── │
│                                                         │
│   Step 1              Step 2              Step 3        │
│   ┌────────┐          ┌────────┐          ┌────────┐   │
│   │Landing │ ──────▶  │Nav CTA │ ──────▶  │Contact │   │
│   │pageView│          │nav_    │          │contact_│   │
│   │path=/de│          │click   │          │submit  │   │
│   └────────┘          └────────┘          └────────┘   │
│       │                   │                   │         │
│       └───────────────────┴───────────────────┘         │
│                    device_id                            │
│                                                         │
│   User must complete ALL steps (same device)            │
│   Order doesn't matter within date range                │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

### Query-Änderung

```sql
-- Vorher (nur event_name + path)
WHERE event_name = 'nav_click' AND path = '/de'

-- Nachher (mit event_data filter)
WHERE event_name = 'nav_click' 
  AND path LIKE '/de%'                        -- starts_with
  AND event_data->>'target' = 'contact'       -- event_data filter
```

---

## Sofort-Workaround

Bis zur Implementierung kann masem.at folgenden Funnel testen:
- Step 1: `pageView`, Path = `/de` (falls starts_with geht, sonst exakt `/de`)
- Step 2: `nav_click`, Value = leer (matcht alle nav_clicks)

Das zeigt zumindest ob die Funnel-Query-Engine grundsätzlich funktioniert.

---

## Aufwandsschätzung

| Komponente | Status | Aufwand |
|------------|--------|---------|
| Query-Engine (`funnel.ts`) | ✅ Unterstützt bereits `event_data` | — |
| FunnelEditor: event_data UI | ❌ Fehlt | ~2h |
| FunnelEditor: Path-Matching Dropdown | ❌ Fehlt | ~1h |
| Examples Panel (Collapsible + Tabs) | ❌ Fehlt | ~2h |
| SVG Tree Components (3 Tabs) | ❌ Fehlt | ~3h |
| **Gesamt** | | **~8h** |

---

## Verknüpfte Issues

- BUG-analytics-tracker-coverage.md → ✅ GEFIXT (beforeInteractive → afterInteractive)
- FIX-SPEC-tracker-spa-route-change.md → ✅ Dokumentiert
- TASK-project-tracker-integration.md → ✅ ChainSights erledigt

---

## Akzeptanzkriterien

### Funktionalität
- [ ] FunnelEditor zeigt optional event_data Key/Value Felder pro Step
- [ ] Funnel-Query matcht korrekt mit event_data Filter
- [ ] Path-Matching unterstützt mindestens `starts_with` (Dropdown: exact/starts_with)
- [ ] Bestehende Funnels ohne event_data-Filter funktionieren weiterhin
- [ ] masem.at "Main Conversion" Funnel ist korrekt konfigurierbar

### Visual Examples Panel
- [ ] Collapsible "Examples" Panel unterhalb der Steps (default: collapsed)
- [ ] Tab 1: PageView + Path – zeigt exact vs starts_with mit ✓/✗ Beispielen
- [ ] Tab 2: Custom Event – zeigt event_name → event_data → match Flow
- [ ] Tab 3: Multi-Step Flow – zeigt device_id Chain über mehrere Steps
- [ ] SVGs als React-Komponenten (nicht statische Bilder)
- [ ] Konsistentes Dark-Theme Styling mit bestehendem Editor
