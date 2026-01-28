# Handoff: Stripe Webhook → Analytics Integration

**Von:** masemIT Analytics Team
**An:** ChainSights Team, tellingCube Team
**Datum:** 2026-01-28
**Priorität:** Medium (Blocker für Conversion Funnel)

---

## Was wird gebraucht?

Wir bauen Conversion Funnels im Analytics Dashboard. Dafür brauchen wir `checkout_complete` Events von euren Stripe Webhooks.

**Aktuell:** Stripe Webhook verarbeitet Zahlung → Event bleibt lokal
**Neu:** Stripe Webhook verarbeitet Zahlung → Event wird auch an Analytics gesendet

---

## Was ihr tun müsst

### 1. Findet euren Stripe Webhook Handler

| Projekt | Vermutlicher Pfad |
|---------|-------------------|
| ChainSights | `src/app/api/webhooks/stripe/route.ts` |
| tellingCube | `app/api/webhooks/stripe/route.ts` oder `app/api/stripe/webhook/route.ts` |

### 2. Fügt diesen Code nach erfolgreicher Zahlung hinzu

```typescript
// Nach der erfolgreichen Verarbeitung von 'checkout.session.completed'
// (also NACH eurem bestehenden Code, nicht stattdessen!)

if (event.type === 'checkout.session.completed') {
  const session = event.data.object;

  // ... euer bestehender Code ...

  // NEU: Event an Analytics senden
  try {
    await fetch('https://analytics.masem.at/api/events', {
      method: 'POST',
      headers: { 'Content-Type': 'text/plain' },
      body: JSON.stringify({
        type: 'checkout_complete',
        project_id: 'chainsights',  // ÄNDERN für tellingCube: 'telling-cube'
        timestamp: new Date().toISOString(),
        path: '/checkout/success',
        event_data: {
          product: session.metadata?.product || 'unknown',
          price_eur: (session.amount_total || 0) / 100,
          dao_name: session.metadata?.dao_name || null,  // Nur ChainSights
          stripe_session_id: session.id,
        }
      })
    });
    console.log('[Analytics] checkout_complete event sent');
  } catch (analyticsError) {
    // Nicht kritisch - Zahlung war erfolgreich, nur Analytics fehlt
    console.error('[Analytics] Failed to send event:', analyticsError);
  }
}
```

### 3. Wichtige Hinweise

| Aspekt | Details |
|--------|---------|
| **Fehler abfangen** | Der Analytics-Call darf die Webhook-Response NICHT blockieren! Stripe erwartet schnelle 200 OK. |
| **project_id** | ChainSights: `'chainsights'`, tellingCube: `'telling-cube'` |
| **Content-Type** | Muss `text/plain` sein (CORS-Optimierung) |
| **price_eur** | Stripe gibt Cents, also `/100` für Euro |

---

## Testen

1. Macht einen Test-Checkout (Stripe Test Mode)
2. Schaut in https://analytics.masem.at/dashboard/chainsights (oder /telling-cube)
3. Sucht nach Event-Typ `checkout_complete`

**Oder fragt mich** - ich kann in der DB nachschauen ob Events ankommen.

---

## Zeitrahmen

- **Nicht super-dringend**, aber wir können den Conversion Funnel erst fertig bauen wenn das steht
- Idealerweise diese Woche

---

## Fragen?

Meldet euch bei Mario oder direkt im Analytics-Projekt.

---

*Erstellt von Paige (Tech Writer) im Rahmen des Analytics Conversion Features Projekts*
