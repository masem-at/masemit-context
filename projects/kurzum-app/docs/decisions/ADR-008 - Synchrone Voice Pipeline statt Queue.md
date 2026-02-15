# ADR-008: Synchrone Voice Pipeline statt Queue

**Datum:** 2026-02-15
**Status:** Entschieden

## Kontext

Die Voice-Verarbeitung in kurzum.app durchlaeuft drei KI-Stufen:
1. **STT:** Audio-Upload an Mistral Voxtral (Transkription)
2. **LLM:** Transkript an Mistral Small (strukturierte Zusammenfassung)
3. **Assignment:** Transkript + Projektliste an Mistral Small (Projekt-Zuordnung)

Zwei Architektur-Ansaetze wurden evaluiert:
- **Synchron:** Alle Stufen in einem HTTP-Request (`app/api/voice/route.ts`)
- **Asynchron:** Upload sofort bestaetigen, Verarbeitung via Message Queue (z.B. Upstash QStash, Vercel Background Functions)

## Entscheidung

**Synchrone Verarbeitung in einem Request.** Upload, STT, LLM-Summarization und Assignment werden sequenziell in `POST /api/voice` ausgefuehrt.

### Implementierung (`app/api/voice/route.ts`)

```
Request → Auth → Upload → DB Insert (status: "uploaded")
       → STT (Voxtral) → DB Update (status: "transcribed")
       → LLM Summary (Mistral Small) → DB Update (status: "done")
       → Assignment (Mistral Small, best-effort) → DB Update (projectId)
       → Response (vollstaendiges Ergebnis)
```

**Fehlerbehandlung als Wasserfall:**
- STT-Fehler → `status: "error"`, Audio bleibt gespeichert
- LLM-Fehler → `status: "partial"`, Transkript bleibt erhalten
- Assignment-Fehler → Silent fail, `status` bleibt "done", Nachricht geht in Inbox

## Begruendung

| Kriterium | Synchron | Queue/Async |
|-----------|---------|-------------|
| Gesamt-Latenz | ~3.7s (697ms STT + 2106ms LLM + 912ms Assign) | Aehnlich + Queue-Overhead |
| Vercel Timeout | 60s (Pro Plan) — 3.7s ist 6% davon | Nicht relevant |
| UX | Sofortiges Ergebnis nach Upload | Polling/WebSocket noetig |
| Komplexitaet | 1 Datei, linearer Flow | Queue-Setup, Retry-Logik, Status-Polling |
| Fehler-Transparenz | Direkte Fehlermeldung an User | Asynchrone Fehler schwerer zu kommunizieren |
| Kosten | $0/Monat (kein Queue-Service) | Upstash QStash ab $1/Monat |
| Debugging | Lineare Ausfuehrung, einfaches Tracing | Verteiltes System, schwerer zu debuggen |

### Warum kein Queue?

1. **Latenz innerhalb Budget:** Die gemessene Gesamtlatenz von ~3.7s liegt weit unter dem Vercel-60s-Timeout und dem S1-NFR5-Budget von 5s fuer die reine KI-Verarbeitung.

2. **Kein Skalierungsproblem:** Pilot-Phase mit 5-10 Handwerksbetrieben, geschaetzte Last: max 100 Nachrichten/Tag. Synchrone Verarbeitung skaliert problemlos.

3. **UX-Vorteil:** Der Monteur sieht sofort nach dem Upload die Zusammenfassung und Zuordnung. Kein Warten, kein Reload, kein "Deine Nachricht wird verarbeitet"-Zustand.

4. **Graceful Degradation:** Jede Stufe hat eigene Fehlerbehandlung. STT-Fehler verhindert nicht den Upload, LLM-Fehler nicht das Transkript. Assignment ist best-effort und failet silent in die Inbox.

## Konsequenzen

### Positiv
- Einfache Architektur: 1 API-Route, linearer Flow, kein verteiltes System
- Sofortiges Ergebnis fuer den Nutzer
- Einfaches Debugging und Monitoring

### Negativ/Risiko
- Bei Mistral-API-Ausfall blockiert der gesamte Request (mitigiert durch Timeout)
- Keine Retry-Logik bei transientem Fehler (mitigiert durch manuelle Neuaufnahme)
- Skaliert nicht unbegrenzt bei gleichzeitigen Uploads (akzeptabel fuer Pilot)

### Spaeter evaluieren
- Queue-basierte Verarbeitung wenn Pilot-Last >500 Nachrichten/Tag uebersteigt
- Background Functions wenn Mistral-Latenz signifikant steigt
- WebSocket-Updates fuer Real-Time-Dashboard (Sprint 3+)
