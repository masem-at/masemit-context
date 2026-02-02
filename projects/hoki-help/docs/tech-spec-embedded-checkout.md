# Tech Spec: Stripe Embedded Checkout + PayPal

**Datum:** 2026-02-02
**Status:** Implementiert
**Priorität:** High (Conversion Funnel Optimierung)

---

## Problem

Nutzer brechen den Spenden-Flow ab, nachdem sie auf die externe Stripe Checkout Seite weitergeleitet werden. Der Kontextwechsel ("Wo bin ich?") unterbricht die emotionale Verbindung und senkt die Conversion Rate.

## Lösung

Migration von Stripe Hosted Checkout (Redirect) zu **Stripe Embedded Checkout** (inline). Zusätzlich **PayPal als Zahlungsmethode** über Stripe aktivieren.

---

## Aktueller Flow

```
DonationWidget → POST /api/checkout → session.url → window.location.href (REDIRECT)
→ Stripe Hosted Page → Redirect /danke?amount=X
```

## Neuer Flow

```
DonationWidget → POST /api/checkout → client_secret
→ Embedded Checkout Form rendert INLINE auf hoki.help
→ onComplete Callback → /danke?session_id=X
```

---

## Scope

### In Scope

- Stripe Embedded Checkout ersetzen (inline statt Redirect)
- PayPal als Payment Method aktivieren (via Stripe Dashboard)
- Return-URL Handling anpassen
- Danke-Seite: Session-Daten per API statt Query-Params
- Analytics-Events bleiben unverändert (Webhook-basiert)

### Out of Scope

- Separate PayPal SDK Integration (nicht nötig, läuft über Stripe)
- Änderungen am Webhook Handler
- Änderungen am Spendenbarometer oder KV-Logik

---

## Technische Änderungen

### 1. Neue Dependencies

```bash
npm install @stripe/stripe-js @stripe/react-stripe-js
```

### 2. Neues File: `src/lib/stripe-client.ts`

Client-seitige Stripe-Initialisierung mit Public Key.

```typescript
import { loadStripe } from '@stripe/stripe-js'

const stripePromise = loadStripe(process.env.NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY!)

export { stripePromise }
```

**Env-Variable benötigt:** `NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY`

### 3. Änderung: `src/app/api/checkout/route.ts`

| Vorher | Nachher |
|--------|---------|
| Kein `ui_mode` | `ui_mode: 'embedded'` |
| `success_url` + `cancel_url` | `return_url` mit `{CHECKOUT_SESSION_ID}` |
| Return: `{ url: session.url }` | Return: `{ clientSecret: session.client_secret }` |

**Wichtig:**
- `success_url` und `cancel_url` entfallen bei `ui_mode: 'embedded'`
- Stattdessen `return_url` mit Placeholder: `/danke?session_id={CHECKOUT_SESSION_ID}`
- `session.client_secret` wird statt `session.url` zurückgegeben

### 4. Änderung: `src/components/DonationWidget.tsx`

**Neuer Flow nach Submit:**

1. POST an `/api/checkout` → empfängt `clientSecret`
2. State-Wechsel: Betrag/Toggle/Checkbox wird ausgeblendet
3. `<EmbeddedCheckoutProvider>` + `<EmbeddedCheckout>` wird gerendert
4. Stripe rendert Payment Form inline (Karte, PayPal, Link, etc.)
5. Nutzer bezahlt direkt auf hoki.help
6. `onComplete` → Redirect zu `/danke?session_id=...`

**Konzeptuelle Struktur:**

```tsx
// Zustand: Formular
if (!clientSecret) {
  return <BetragsAuswahl ... />
}

// Zustand: Checkout
return (
  <EmbeddedCheckoutProvider stripe={stripePromise} options={{ clientSecret }}>
    <EmbeddedCheckout />
  </EmbeddedCheckoutProvider>
)
```

**Zurück-Option:** Button "Betrag ändern" der `clientSecret` zurücksetzt.

### 5. Änderung: `src/app/danke/page.tsx` + `ThankYouContent.tsx`

**Vorher:** Query-Params `?amount=X&monthly=Y`
**Nachher:** Query-Param `?session_id=X`

Neue Server-Side Logik:
1. `session_id` aus Query-Params lesen
2. Stripe API: `stripe.checkout.sessions.retrieve(session_id)` aufrufen
3. `amount`, `isMonthly`, `isAnonymous` aus Session extrahieren
4. An `ThankYouContent` übergeben (wie bisher)

**Vorteil:** Daten kommen verifiziert von Stripe, nicht manipulierbar via URL.

### 6. Stripe Dashboard: PayPal aktivieren

Manuelle Konfiguration (kein Code nötig):

1. Stripe Dashboard → Settings → Payment Methods
2. PayPal aktivieren und Stripe-Account mit PayPal verknüpfen
3. Embedded Checkout zeigt PayPal automatisch als Option

---

## Env-Variablen

| Variable | Neu? | Beschreibung |
|----------|------|--------------|
| `STRIPE_SECRET_KEY` | Nein | Server-Side Key (existiert) |
| `STRIPE_WEBHOOK_SECRET` | Nein | Webhook Verifizierung (existiert) |
| `NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY` | **Ja** | Client-Side Publishable Key |
| `NEXT_PUBLIC_BASE_URL` | Nein | Base URL (existiert) |

---

## Betroffene Dateien

| Datei | Aktion |
|-------|--------|
| `package.json` | Dependencies hinzufügen |
| `src/lib/stripe-client.ts` | **Neu** erstellen |
| `src/app/api/checkout/route.ts` | `ui_mode: 'embedded'`, Return `clientSecret` |
| `src/components/DonationWidget.tsx` | Embedded Checkout Form statt Redirect |
| `src/app/danke/page.tsx` | Session-ID statt Query-Params |
| `src/components/ThankYouContent.tsx` | Props bleiben gleich |
| `.env` / Vercel Env | `NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY` |
| Stripe Dashboard | PayPal Payment Method aktivieren |

---

## Nicht betroffen

| Datei | Warum |
|-------|-------|
| `src/app/api/webhook/route.ts` | Webhook bleibt identisch |
| `src/lib/stripe.ts` | Server-Client unverändert |
| `src/lib/kv.ts` | Spendenbarometer unverändert |
| Analytics Integration | Webhook-basiert, kein Frontend-Bezug |

---

## Risiken

| Risiko | Mitigation |
|--------|------------|
| `custom_fields` (Geburtsdatum) evtl. nicht in Embedded Mode | Prüfen in Stripe Docs, ggf. separates Feld im DonationWidget |
| PayPal unterstützt kein `subscription` mode | Prüfen - ggf. PayPal nur für Einmalspenden, Karte für beides |
| NEXT_PUBLIC Key vergessen zu setzen | Build schlägt fehl, sofort sichtbar |

---

## Testen

1. `NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY` in `.env.local` setzen
2. `npm run dev`
3. Spende starten → Embedded Form muss inline erscheinen
4. Test-Karte `4242 4242 4242 4242` verwenden
5. Prüfen: Danke-Seite zeigt korrekte Daten
6. Prüfen: Webhook feuert, Analytics-Event kommt an
7. PayPal: Nur testbar wenn im Stripe Dashboard aktiviert

---

## Siehe auch

- [Stripe Embedded Checkout Docs](https://docs.stripe.com/checkout/embedded/quickstart)
- [Analytics Integration](./analytics-integration.md)
