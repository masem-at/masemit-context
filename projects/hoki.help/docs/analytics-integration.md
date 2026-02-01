# Analytics Integration: hoki-help → analytics.masem.at

**Datum:** 2026-01-28
**Status:** Implementiert
**Datei:** `src/app/api/webhook/route.ts`

---

## Übersicht

hoki-help sendet `checkout_complete` Events an das zentrale Analytics-Dashboard unter `analytics.masem.at`. Diese Events ermöglichen Conversion Funnel Analyse für Spenden.

---

## Event-Typen

### 1. Einmalige Spenden

**Trigger:** Stripe Event `checkout.session.completed`

```json
{
  "type": "checkout_complete",
  "project_id": "hoki-help",
  "timestamp": "2026-01-28T10:30:00.000Z",
  "path": "/checkout/success",
  "event_data": {
    "product": "donation",
    "donation_type": "one_time",
    "price_eur": 25.00,
    "is_anonymous": false,
    "stripe_session_id": "cs_xxx"
  }
}
```

### 2. Wiederkehrende Spenden (Monatlich)

**Trigger:** Stripe Event `invoice.paid`

```json
{
  "type": "checkout_complete",
  "project_id": "hoki-help",
  "timestamp": "2026-01-28T10:30:00.000Z",
  "path": "/checkout/success",
  "event_data": {
    "product": "donation",
    "donation_type": "monthly",
    "price_eur": 10.00,
    "stripe_invoice_id": "in_xxx"
  }
}
```

---

## Technische Details

| Aspekt | Wert |
|--------|------|
| **Endpoint** | `https://analytics.masem.at/api/events` |
| **Method** | `POST` |
| **Content-Type** | `text/plain` (CORS-Optimierung) |
| **project_id** | `hoki-help` |
| **Error Handling** | Fire-and-forget (non-blocking) |

---

## Event-Daten Mapping

| Feld | Quelle | Beschreibung |
|------|--------|--------------|
| `donation_type` | `session.metadata.donationType` | `one_time` oder `monthly` |
| `price_eur` | `amount_total / 100` | Betrag in Euro |
| `is_anonymous` | `session.metadata.isAnonymous` | Anonyme Spende |
| `stripe_session_id` | `session.id` | Checkout Session ID |
| `stripe_invoice_id` | `invoice.id` | Invoice ID (nur monthly) |

---

## Funnel-Struktur im Dashboard

```
Page Visit → Donate Click → Checkout Started → checkout_complete
```

Das `checkout_complete` Event schließt den Conversion Funnel.

---

## Testen

1. Test-Spende im Stripe Test Mode durchführen
2. Dashboard prüfen: `https://analytics.masem.at/dashboard/hoki-help`
3. Event-Typ `checkout_complete` suchen

**Logs prüfen:**
```
[Analytics] checkout_complete event sent
[Analytics] recurring checkout_complete event sent
```

---

## Fehlerbehandlung

Analytics-Fehler blockieren NICHT die Webhook-Verarbeitung:
- Spende wird trotzdem erfasst (`incrementDonation`)
- Stripe erhält `200 OK`
- Fehler werden geloggt: `[Analytics] Failed to send event:`

---

## Siehe auch

- [Handoff-Dokument](./_masemIT/handoff-stripe-webhook-analytics.md)
- [Stripe Webhook Handler](../src/app/api/webhook/route.ts)
