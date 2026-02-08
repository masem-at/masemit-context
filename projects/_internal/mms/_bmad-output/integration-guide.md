# MMS Integration Guide

> For Sempre: Step-by-step guide to migrate masemIT projects to use the MMS Donations API.

---

## Prerequisites

Before starting any migration:

1. **API Keys generieren** — Für jedes Projekt einen Key in der MMS-Datenbank anlegen
2. **MMS API erreichbar** — `https://api.masem.at/health` sollte `{"status":"ok"}` antworten
3. **Env-Vars vorbereiten** — `MMS_API_KEY` und `MMS_API_URL` für jedes Projekt

---

## API Key Setup

### Key generieren (einmalig pro Projekt)

```bash
# In mms-api Verzeichnis
npx tsx src/db/seed.ts
```

Oder manuell via SQL (Neon Console):

```sql
INSERT INTO api_keys (project, key_hash, key_prefix, name, scopes, is_active)
VALUES (
  'hoki_help',
  -- SHA-256 hash des Keys
  encode(sha256('mms_hoki_DEIN_GEHEIMER_KEY'::bytea), 'hex'),
  'mms_hoki',
  'HoKi Help Production',
  ARRAY['donations:read', 'donations:write'],
  true
);
```

### Key Format

```
mms_{project}_{random}

Beispiele:
- mms_hoki_a1b2c3d4e5f6g7h8
- mms_tellingcube_x9y8z7w6v5u4
- mms_chainsights_m1n2o3p4q5r6
```

---

## Migration pro Projekt

### 1. hoki.help

**Aktuell:** Lokale `donations` Tabelle in hoki.help DB

**Schritte:**

1. **Env-Vars hinzufügen:**
   ```env
   MMS_API_URL=https://api.masem.at
   MMS_API_KEY=mms_hoki_xxxxx
   ```

2. **API Client erstellen:**
   ```typescript
   // lib/mms-client.ts
   const MMS_API_URL = process.env.MMS_API_URL
   const MMS_API_KEY = process.env.MMS_API_KEY

   export async function recordDonation(data: {
     target_id?: string  // Optional: wenn angegeben → Direct, sonst → Distributed
     product_price_cents: number  // Produktpreis, MMS berechnet Spendenbetrag
     donation_type: 'direct' | 'revenue_share'
     reference?: string
     donor_note?: Record<string, unknown>  // JSONB, z.B. { first_name, is_anonymous }
     donated_at: Date
   }) {
     const res = await fetch(`${MMS_API_URL}/v1/donations`, {
       method: 'POST',
       headers: {
         'Content-Type': 'application/json',
         'x-api-key': MMS_API_KEY!,
       },
       body: JSON.stringify({
         ...data,
         source_project: 'hoki_help',
         currency: 'EUR',
         donated_at: data.donated_at.toISOString(),
       }),
     })

     if (!res.ok) {
       throw new Error(`MMS API Error: ${res.status}`)
     }

     return res.json()
   }
   ```

   **Wichtig:** MMS berechnet den Spendenbetrag aus `product_price_cents` × Projekt-Prozentsatz (z.B. 100% für hoki_help).

3. **Donation-Flow anpassen:**
   - Wo bisher in lokale DB geschrieben wurde → `recordDonation()` aufrufen
   - Stripe Webhook Handler anpassen (falls vorhanden)

4. **Target ID ermitteln:**
   ```bash
   curl https://api.masem.at/v1/donation-targets \
     -H "x-api-key: mms_hoki_xxxxx"
   ```
   Die `id` von "HoKi NÖ" als Konstante speichern.

5. **Lokale Tabelle deprecaten:**
   - Reads zuerst auf MMS umstellen
   - Writes auf MMS umstellen
   - Nach Verifizierung: lokale Tabelle droppen

---

### 2. ChainSights

**Aktuell:** Crypto-Donations via Coinbase Commerce

**Schritte:**

1. **Env-Vars** (wie oben, mit `mms_chainsights_xxxxx`)

2. **Coinbase Webhook anpassen:**
   ```typescript
   // Bei erfolgreicher Zahlung
   await recordDonation({
     target_id: HOKI_TARGET_ID,  // Direct donation
     product_price_cents: convertCryptoToEurCents(payment),
     donation_type: 'direct',
     reference: `coinbase_${payment.id}`,
     donated_at: new Date(payment.created_at),
   })
   ```

3. **Revenue Share (bei Subscriptions):**
   ```typescript
   // Bei Subscription-Renewal — MMS berechnet automatisch den %-Anteil
   await recordDonation({
     // Kein target_id → Distributed per target_distribution Tabelle
     product_price_cents: subscriptionPriceCents,
     donation_type: 'revenue_share',
     reference: `stripe_sub_${subscriptionId}_period_${periodId}`,
     donated_at: new Date(),
   })
   // MMS berechnet: subscriptionPriceCents × 5% (ChainSights Default)
   ```

---

### 3. tellingCube

**Schritte:** Analog zu ChainSights

- `source_project: 'tellingcube'`
- Revenue Share bei Premium-Subscriptions

---

### 4. masem.at (Verantwortungsseite)

**Neu:** Konsumiert den Summary Endpoint (API Key erforderlich — Auth-by-Default Policy)

**Implementierung:**

```typescript
// lib/donations.ts
export async function getDonationSummary() {
  const res = await fetch(
    'https://api.masem.at/v1/donations/summary',
    {
      headers: { 'x-api-key': process.env.MMS_API_KEY! },
      next: { revalidate: 900 } // 15 min ISR
    }
  )

  if (!res.ok) {
    throw new Error('Failed to fetch donation summary')
  }

  return res.json()
}
```

```tsx
// app/verantwortung/page.tsx
export default async function VerantwortungPage() {
  const { data } = await getDonationSummary()

  return (
    <section>
      <h1>Unsere Verantwortung</h1>

      {data.map(item => (
        <div key={item.target.id}>
          <h2>{item.target.name}</h2>
          <p>
            Gespendet: {formatCurrency(item.total_donated_cents / 100)}
          </p>
          <p>{item.donation_count} Spenden</p>
        </div>
      ))}
    </section>
  )
}
```

**API Key erforderlich** — Auth-by-Default Policy (seit 2026-02-04).

---

## Checkliste pro Migration

- [ ] Env-Vars gesetzt (`MMS_API_URL`, `MMS_API_KEY`)
- [ ] API Client implementiert
- [ ] Target ID als Konstante gespeichert
- [ ] Donation-Flow auf MMS umgestellt
- [ ] Lokale Reads verifiziert (Summen stimmen)
- [ ] Lokale Tabelle deprecated/entfernt
- [ ] Production deployed & getestet

---

## Troubleshooting

### 401 Unauthorized
- API Key korrekt? (mit Prefix `mms_{project}_`)
- Key in DB aktiv? (`is_active = true`)
- Scopes ausreichend? (z.B. `donations:write` für POST)

### 429 Rate Limited
- Public Endpoint: 100 req/min
- Mit Key: höheres Limit, aber trotzdem throttled

### 400 Validation Error
- `target_id` muss existierende UUID sein
- `amount_cents` muss positive Integer sein
- `donated_at` muss gültiges Datum sein

---

## Docs Endpoint (für Agents)

Sobald implementiert, können Agents in anderen Projekten die API-Spec laden:

```
WebFetch: https://api.masem.at/docs/donations
```

Das Markdown enthält alle Endpoints, Schemas und Beispiele — optimiert für LLM-Consumption.
