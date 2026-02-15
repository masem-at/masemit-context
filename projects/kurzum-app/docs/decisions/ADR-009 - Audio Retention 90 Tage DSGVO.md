# ADR-009: Audio Retention 90 Tage (DSGVO)

**Datum:** 2026-02-15
**Status:** Entschieden

## Kontext

kurzum.app speichert Audio-Aufnahmen von Handwerkern (Sprachnachrichten von der Baustelle). Diese Aufnahmen enthalten personenbezogene Daten — Stimme ist ein biometrisches Merkmal, Inhalte koennen Namen, Adressen und Projektdetails enthalten.

Die DSGVO (Art. 5 Abs. 1 lit. e — Speicherbegrenzung) verlangt, dass personenbezogene Daten nur so lange gespeichert werden, wie es fuer den Verarbeitungszweck erforderlich ist. Der FFG-Antrag (Antragsnr. 69168884) definiert "DSGVO-by-Design" als Projektanforderung.

## Entscheidung

**90 Tage Aufbewahrungsfrist fuer Audio-Dateien**, danach automatische Loeschung. Transkript und Zusammenfassung bleiben dauerhaft erhalten.

### Implementierung

**Datenbank-Schema (`lib/db/schema.ts`):**
- Column `audio_deleted_at` (Timestamp, nullable) in der `voice_messages` Tabelle
- Markiert den Zeitpunkt der Audio-Loeschung (Soft-Delete-Tracking)
- Audio-URL bleibt als Referenz, Datei wird aus dem Storage geloescht

**Retention-Logik (geplant):**
1. Cron-Job/Scheduled Function prueft taeglich: `WHERE created_at < NOW() - INTERVAL '90 days' AND audio_deleted_at IS NULL`
2. Audio-Datei aus Vercel Blob Storage loeschen
3. `audio_deleted_at = NOW()` setzen
4. `audio_url` optional auf Platzhalter setzen

**Was bleibt erhalten:**
- Transkript (Text, kein biometrisches Merkmal)
- Zusammenfassung (KI-generiert, aggregiert)
- Metadaten (Zeitstempel, Dauer, Zuordnung)
- Audit-Trail (wann wurde Audio geloescht)

## Begruendung

| Kriterium | 90 Tage | Unbegrenzt | Sofort loeschen |
|-----------|---------|-----------|----------------|
| DSGVO-Konformitaet | Verhaeltnismaessig | Problematisch | Maximal konform |
| Nutzererwartung | Audio nachhoerbar fuer ~3 Monate | "Warum habt ihr noch meine Aufnahme von vor 2 Jahren?" | "Wo ist meine Aufnahme hin?" |
| Debugging/QA | Ausreichend fuer Pilot-Phase | Unbegrenzt | Nicht moeglich |
| Speicherkosten | Begrenzt | Wachsend | Minimal |
| FFG-Audit | Nachweisbar: definierter Loeschzyklus | Kein Loeschkonzept | Keine Forschungsdaten |

### Warum 90 Tage?

1. **Praxisnaehe:** Ein Bauprojekt laeuft typischerweise 2-6 Monate. 90 Tage deckt den Grossteil der aktiven Projektphase ab, in der ein Nachhoeren der Originalnachricht sinnvoll sein koennte.

2. **Streitigkeiten:** Bei Meinungsverschiedenheiten zwischen Monteur und Chef ueber den Inhalt einer Nachricht kann das Audio als Beleg dienen. 90 Tage reichen fuer die meisten internen Klaerungen.

3. **Forschungszweck:** Waehrend der FFG-Pilotphase ermoeglicht die 90-Tage-Frist die nachtraegliche Analyse von STT-Fehlern und Pipeline-Verbesserungen.

4. **DSGVO-Verhaeltnismaessigkeit:** Weder zu kurz (Nutzer verliert Zugang zu frueh) noch zu lang (unnoetige Datenhaltung). Vergleichbar mit Messenger-Apps die Medien nach X Tagen loeschen.

## Konsequenzen

### Positiv
- DSGVO-konforme Speicherbegrenzung dokumentiert und implementiert
- Speicherkosten begrenzt und vorhersehbar
- FFG-Audit: Definiertes Loeschkonzept als R&D-Nachweis

### Negativ/Risiko
- Cron-Job/Cleanup-Mechanismus muss noch implementiert werden (Tech Debt)
- Nutzer kann Audio nach 90 Tagen nicht mehr anhoeren (kein Undo)
- Transkript-Fehler koennen nach Loeschung nicht mehr anhand des Originals verifiziert werden

### Spaeter evaluieren
- Nutzer-konfigurierbare Retention (30/60/90/180 Tage) als Premium-Feature
- Explizites User-Consent fuer laengere Aufbewahrung (DSGVO Art. 6 Abs. 1 lit. a)
- Verschluesselung-at-Rest fuer Audio-Dateien waehrend der Aufbewahrungsfrist
