# ESG Plan C – Validierung: Interview Pollmann (Zulieferer)

> **Projekt:** ESG Reporting SaaS für KMU-Zulieferer
> **Dokument:** Erste Marktvalidierung – Qualitatives Interview
> **Datum:** 01.02.2026
> **Interviewpartner:** Mario Sempers Bruder, Mitarbeiter bei Pollmann (Automotive-Zulieferer, ~1.300 MA)
> **Methode:** Informelles Interview via Chat (5 Fragen + 3 Follow-ups)

---

## 1. Kontext

Pollmann ist ein österreichischer Automotive-Zulieferer mit ca. 1.300 Mitarbeitern, der Komponenten für OEMs wie BMW und Magna liefert. Das Unternehmen fällt in die Kategorie "mittelständischer Tier-1/Tier-2 Zulieferer" – exakt die Zielgruppe für ein ESG-Zulieferer-Tool.

Das Interview dient als erster qualitativer Validierungsschritt, bevor Förderanträge (FFG, aws) gestellt oder ein MVP entwickelt wird.

---

## 2. Ergebnisse

### 2.1 Existiert das Problem? ✅ Ja

OEMs wie BMW fragen **seit ca. 2 Jahren** aktiv ESG- und Nachhaltigkeitsdaten bei ihren Zulieferern an. Der Druck kommt nicht aus der Regulatorik (Taxonomie-Verschiebung wird wahrgenommen), sondern direkt von den Kunden – unabhängig vom Gesetzgeber.

> **Takeaway:** Das Problem ist real und aktuell. Es handelt sich nicht um ein "kommt vielleicht irgendwann"-Szenario, sondern um bestehende, wiederkehrende Anforderungen.

### 2.2 Wie groß ist der Pain? ✅ Hoch

- ESG/Nachhaltigkeit wird intern als **"lästige Pflicht"** gesehen.
- Es gibt eine **eigene, hauptberuflich** damit beschäftigte Person.
- CO2 in **Produktion und Lieferketten** spielt die zentrale Rolle.

> **Takeaway:** Eine Vollzeitstelle für eine als lästig empfundene Aufgabe = realer Kostenfaktor (geschätzt 50–70k €/Jahr all-in) und echtes Automatisierungspotenzial.

### 2.3 Welche Formate kommen? ⚠️ Heterogen

- **Jeder Kunde macht es "a bissl anders"** – keine einheitlichen Standards.
- Frequenz: ca. **1x pro Jahr** pro Kunde.
- Bei mehreren OEM-Kunden summiert sich das.

> **Takeaway:** Multi-Format-Fähigkeit ist ein Muss. Ein Tool, das verschiedene Kundenformate bedienen kann und ein standardisiertes Basisprofil als Ausgangspunkt bietet, trifft den Nerv.

### 2.4 Taxonomie-Verschiebung als Doppelsignal

Die Verschiebung der EU-Taxonomie wird wahrgenommen und führt dazu, dass das Thema intern etwas weniger Dringlichkeit hat. **Aber:** Die OEM-Anfragen kommen trotzdem – die Großkunden treiben ihre Lieferketten-Transparenz unabhängig vom Gesetzgeber voran.

> **Takeaway:** Die Regulatorik ist der Rückenwind, aber der direkte Kundendruck ist der eigentliche Treiber. Das Tool muss den OEM-Bedarf adressieren, nicht primär Compliance-Reporting.

---

## 3. Validierungs-Scorecard

| Hypothese | Status | Evidenz |
|-----------|--------|---------|
| OEMs fordern ESG-Daten von Zulieferern | ✅ Bestätigt | BMW fragt seit ~2 Jahren an |
| Das Thema bindet signifikante Ressourcen | ✅ Bestätigt | Eigene Vollzeitstelle |
| CO2/PCF ist der zentrale Datenpunkt | ✅ Bestätigt | "CO2 in Produktion und Lieferketten" |
| Kundenformate sind heterogen | ✅ Bestätigt | "Jeder Kunde macht's anders" |
| Es gibt Bereitschaft, für Vereinfachung zu zahlen | ⏳ Noch nicht validiert | Kein direkter Preistest durchgeführt |
| Ein standardisiertes Profil wäre hilfreich | ⏳ Noch nicht validiert | Nicht explizit gefragt |

**Gesamt:** 4/6 Hypothesen bestätigt, 2 offen.

---

## 4. Implikationen für den MVP

### Must-haves (aus dem Interview abgeleitet)

1. **PCF-Rechner (Product Carbon Footprint):** CO2 pro Bauteil berechenbar aus Basisdaten (Energieverbrauch, Materialien, Lieferwege).
2. **Multi-Format-Export:** Verschiedene Ausgabeformate für verschiedene Kunden, idealerweise mit einem standardisierten Basisprofil als Kern.
3. **Einfachheit:** Das Tool muss von einer Person bedienbar sein, die das Thema als "lästige Pflicht" sieht – keine ESG-Expertenwissen-Voraussetzung.

### Nice-to-haves

- Fragebogen-Assistent (Schritt-für-Schritt durch Kundenfragebögen)
- Automatische Datenübernahme aus Vorjahren
- Benchmark-Vergleich mit Branchendurchschnitt

---

## 5. Offene Fragen für Follow-up

Nach dem Urlaub des Interviewpartners:

1. **Zugang zur ESG-Person:** Wäre ein kurzes Gespräch (15–20 min) mit der Person möglich, die das bei Pollmann hauptberuflich macht? → Verständnis der tatsächlichen Workflows, Tools, Pain Points.
2. **Konkrete Formate:** Welche Formate/Fragebögen kommen von BMW, Magna etc. konkret? (CDP, EcoVadis, eigene Formate?)
3. **Bestehende Tools:** Nutzt die Person bereits Software dafür oder arbeitet sie mit Excel/manuell?
4. **Zahlungsbereitschaft:** Was wäre ein fairer Preis für ein Tool, das die Arbeit um 50% reduziert?

---

## 6. Nächste Schritte

- [ ] Urlaub abwarten, dann Follow-up-Fragen an den Bruder
- [ ] Zweiten Validierungskontakt (Test-Fuchs / Aerospace) aktivieren
- [ ] Bei 2+ positiven Validierungen: FFG Projekt.Start vorbereiten
- [ ] Breitere Umfrage unter NÖ-Zulieferern planen (mit Referenz auf Pollmann-Validierung)

---

## Anhang: Rohdaten Interview

**Fragen gestellt:**

1. Bekommt ihr von euren großen Kunden (wie BMW, Magna etc.) auch immer öfter Anfragen und Fragebögen zum Thema Nachhaltigkeit und CO2-Fußabdruck eurer Bauteile?
2. Falls ja: Wie aufwändig ist das bei euch? Wer kümmert sich darum und ist das eher ein "nerviges" Thema, das einfach nur Zeit kostet?
3. Stell dir eine super einfache Software vor, die nur für Zulieferer wie euch gemacht ist. Wäre es hilfreich, wenn diese Software euch hilft...
   - a) die Fragebögen der Kunden Schritt für Schritt auszufüllen?
   - b) die wichtigsten Kennzahlen (z.B. den CO2-Verbrauch pro Bauteil) automatisch zu berechnen?
   - c) am Ende ein sauberes, standardisiertes "Lieferanten-Nachhaltigkeitsprofil" als PDF zu erstellen?
4. Was wäre aus deiner Sicht die eine, wichtigste Funktion, die so ein Tool haben müsste?

**Antwort (wörtlich):**

> "Pff, also es gibt bei uns gibt es eine eigene Person. Momentan wird es eher als lästige Pflicht gesehen, da ja die Taxonomie verschoben wurde. Bei uns spielt halt co2 in produktion und lieferketten eine große rolle. Kunden fragen schon seit ca 2 jahren an, vorallem oems zb bmw..."

**Follow-up-Fragen:**

1. Eigene Formate oder Standards wie CDP/EcoVadis? → "Jeder Kunde machts a bissl anders"
2. Wie oft pro Jahr? → "1mal pro jahr denke ich"
3. Hauptjob oder nebenbei? → "Hauptberuflich"

**Zusammenfassung bestätigt:** Ja 👍

---

*Erstellt am 01.02.2026 | masemIT e.U. – ESG Plan C Vorarbeiten*
