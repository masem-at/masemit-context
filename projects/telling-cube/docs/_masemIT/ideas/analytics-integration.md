# Analytics Integration: telling-cube → analytics.masem.at

**Datum:** 2026-01-29
**Status:** Implementiert
**Tracker:** `analytics.masem.at/tracker.js` (client) + `lib/analytics/send-event.ts` (server)

---

## Übersicht

telling-cube sendet Events an `analytics.masem.at` über zwei Kanäle:

- **Client-side:** `window.masemit.track()` via tracker.js für UI-Interaktionen
- **Server-side:** Fire-and-forget POST an `/api/events` für Stripe Webhook Events

---

## Technische Details

| Aspekt | Wert |
|--------|------|
| **Tracker Script** | `https://analytics.masem.at/tracker.js` |
| **API Endpoint** | `https://analytics.masem.at/api/events` |
| **project_id** | `tellingcube` (client) / `telling-cube` (server) |
| **Client API** | `window.masemit?.track(event, props)` |
| **Error Handling** | Fire-and-forget (non-blocking) |

---

## Funnel Event Übersicht

### Awareness → Interest

| Event Name | Trigger | Properties |
|------------|---------|------------|
| `page_view` | Jede Seite | `page`, `referrer`, `utm_source`, `utm_medium`, `utm_campaign` |
| `landing_scroll_depth` | 25%, 50%, 75%, 100% | `depth_percent` |
| `section_view` | Section in Viewport | `section`: hero, examples, problem, use_cases, features, pricing, faq |

### Interest → Exploration

| Event Name | Trigger | Properties |
|------------|---------|------------|
| `examples_click` | Klick auf "Explore Examples" CTA | `source`: hero \| nav |
| `example_card_click` | Klick auf Example Card | `company`, `size`, `sector` |
| `example_view` | Example-Seite geladen | `company`, `size`, `sector` |
| `example_tab_switch` | Tab Wechsel (Finance/Sales/HR) | `company`, `tab` |
| `example_chart_interact` | Chart Hover/Click | `company`, `chart_type` |
| `example_time_spent` | Beim Verlassen | `company`, `duration_seconds` |

### Exploration → Intent

| Event Name | Trigger | Properties |
|------------|---------|------------|
| `pricing_view` | Pricing Section in Viewport | `source`: scroll \| nav_click |
| `pricing_tier_hover` | Hover auf Tier Card | `tier` |
| `generate_cta_click` | "Generate Your Own" Click | `source` |
| `api_docs_click` | API Docs Link Click | - |

### Intent → Conversion

| Event Name | Trigger | Properties |
|------------|---------|------------|
| `checkout_start` | "Get [Tier]" Button Click | `tier`, `price`, `billing`: onetime \| monthly |
| `checkout_email_enter` | Email eingegeben | `tier` |
| `coupon_apply` | Coupon Code eingegeben | `tier`, `coupon_code`, `valid`: true \| false |
| `checkout_complete` | Stripe Success (Server) | `tier`, `price_eur`, `purchase_type`, `stripe_session_id` |
| `checkout_abandon` | Checkout verlassen ohne Kauf | `tier`, `step_reached` |

### Post-Conversion (Activation)

| Event Name | Trigger | Properties |
|------------|---------|------------|
| `login_magic_link_request` | Magic Link angefordert | `source`: checkout \| login_page |
| `login_success` | Erfolgreich eingeloggt | `method`: magic_link |
| `dashboard_view` | Dashboard geladen | `scenario_count`, `tier` |
| `generation_start` | Generation gestartet | `company_size`, `sector`, `years`, `hr_enabled` |
| `generation_complete` | Generation fertig | `company_size`, `sector`, `years`, `duration_seconds`, `event_count` |
| `generation_cancel` | Generation abgebrochen | `company_size`, `reason` |
| `export_download` | CSV/JSON Download | `format`, `view`: finance \| sales \| hr \| events |

### Engagement / Retention

| Event Name | Trigger | Properties |
|------------|---------|------------|
| `return_visit` | User kommt zurück | `days_since_last_visit`, `tier` |
| `scenario_revisit` | Altes Scenario geöffnet | `scenario_id`, `days_since_created` |
| `feature_discovery` | Erstes Mal Feature genutzt | `feature`: hr_domain \| multi_year \| api |

---

## Implementierung

### Client-side Tracking

**Typed Tracking Module:** `lib/analytics/track.ts`

Alle Events haben typisierte Helper-Funktionen:

```typescript
import { trackCheckoutStart, trackExampleView, ... } from '@/lib/analytics/track'

// Beispiel
trackCheckoutStart({ tier: 'starter', price: 29, billing: 'onetime' })
trackExampleView({ company: 'Alpine Bakery GmbH', size: 'startup', sector: 'consumer-staples' })
```

**Tracking Hooks:** `lib/hooks/`

| Hook | Datei | Zweck |
|------|-------|-------|
| `useScrollDepth` | `useScrollDepth.ts` | Scroll-Tiefe 25/50/75/100% |
| `useSectionView` | `useSectionView.ts` | IntersectionObserver für Sections |
| `useTimeSpent` | `useTimeSpent.ts` | Zeit auf Seite beim Verlassen |

**Landing Page Tracker:** `components/landing/LandingTracker.tsx`

Unsichtbare Client-Komponente die Scroll-Depth und Section-Views automatisch trackt.

### Server-side Tracking

**Analytics Helper:** `lib/analytics/send-event.ts`

Sendet `checkout_complete` Events für:
- One-time Käufe (`purchase_type: one_time`)
- Initiale Subscriptions (`purchase_type: subscription_initial`)
- Wiederkehrende Zahlungen (`purchase_type: subscription_recurring`)

---

## Geänderte Dateien

| Datei | Änderung |
|-------|----------|
| `lib/analytics/track.ts` | Typisiertes Client-Tracking-Modul (NEU) |
| `lib/analytics/send-event.ts` | Server-side Event Sender |
| `lib/hooks/useScrollDepth.ts` | Scroll-Tiefe Hook (NEU) |
| `lib/hooks/useSectionView.ts` | Section-View Hook (NEU) |
| `lib/hooks/useTimeSpent.ts` | Time-Spent Hook (NEU) |
| `components/landing/LandingTracker.tsx` | Landing Page Tracker (NEU) |
| `app/page.tsx` | LandingTracker eingebunden |
| `components/landing/Hero.tsx` | `examples_click`, `generate_cta_click`, section ID |
| `components/landing/ExampleCards.tsx` | `example_card_click` |
| `components/landing/PricingSection.tsx` | `checkout_start`, `pricing_tier_hover` |
| `components/landing/ProblemSolution.tsx` | Section ID `problem` |
| `components/landing/UseCasesSection.tsx` | Section ID `use_cases` |
| `components/landing/FeaturesSection.tsx` | Section ID `features` |
| `components/landing/FAQSection.tsx` | Section ID `faq` |
| `app/examples/[slug]/page.tsx` | `example_view`, `example_time_spent`, `export_download` |
| `components/results/ViewNavigation.tsx` | `example_tab_switch` |
| `app/results/[scenarioId]/page.tsx` | `export_download`, `api_docs_click` |
| `app/login/page.tsx` | `login_magic_link_request` |
| `components/dashboard/DashboardContent.tsx` | `dashboard_view` |
| `components/dashboard/GenerationDialog.tsx` | `generation_start` |
| `app/api/stripe/webhook/route.ts` | `checkout_complete` (server) |

---

## Zusätzlich implementierte Dateien (Phase 2)

| Datei | Änderung |
|-------|----------|
| `components/analytics/ReturnVisitTracker.tsx` | Return-Visit Tracking via localStorage (NEU) |
| `components/analytics/FeatureDiscoveryTracker.tsx` | Feature-Discovery Hook via localStorage (NEU) |
| `components/analytics/TrackableChart.tsx` | Chart-Interaktions-Wrapper (NEU) |
| `components/results/HRView.tsx` | `feature_discovery` (hr_domain) |
| `components/dashboard/GenerationDialog.tsx` | `feature_discovery` (multi_year) |
| `app/generate/loading/page.tsx` | `generation_cancel` |
| `app/layout.tsx` | `ReturnVisitTracker` eingebunden |
| `app/results/[scenarioId]/page.tsx` | `scenario_revisit` |
| `app/examples/[slug]/page.tsx` | `TrackableChart` für `example_chart_interact` |
| `app/api/auth/verify/route.ts` | `login_success` (server-side) |
| `app/api/generate/worker/route.ts` | `generation_complete` (server-side) |
| `app/api/stripe/webhook/route.ts` | `checkout_abandon` via `checkout.session.expired` |
| `lib/analytics/send-event.ts` | Generischer Server-Event-Sender + 3 neue Event-Typen |

---

## Nicht implementierbar (Stripe Hosted Checkout)

| Event | Grund |
|-------|-------|
| `checkout_email_enter` | Stripe Hosted Checkout -- kein Zugriff auf Eingabefelder |
| `coupon_apply` | Stripe Hosted Checkout -- kein Zugriff auf Coupon-Eingabe |

Diese Events erfordern Stripe Elements (eigenes Checkout-UI) statt Hosted Checkout.

---

## Automatisch via tracker.js

| Event | Status |
|-------|--------|
| `page_view` | Automatisch von tracker.js gesendet (data-project="tellingcube") |

---

## Stripe Webhook Konfiguration

Folgende Events müssen im Stripe Dashboard unter Webhook-Einstellungen aktiviert sein:

- `checkout.session.completed` ✅
- `customer.subscription.created` ✅
- `customer.subscription.updated` ✅
- `customer.subscription.deleted` ✅
- `invoice.paid` ✅
- `invoice.payment_failed` ✅
- **`checkout.session.expired`** ⚠️ NEU -- für `checkout_abandon` Tracking

---

## Testen

1. Landing Page öffnen → `section_view` Events für sichtbare Sections
2. Runterscrollen → `landing_scroll_depth` bei 25%, 50%, 75%, 100%
3. Example Card klicken → `example_card_click` + `example_view`
4. Tab wechseln → `example_tab_switch`
5. Seite verlassen → `example_time_spent`
6. Pricing Tier hovern → `pricing_tier_hover`
7. Checkout starten → `checkout_start`
8. Dashboard öffnen → `dashboard_view`
9. Generation starten → `generation_start`
10. Export downloaden → `export_download`
11. Chart klicken (Example Page) → `example_chart_interact`
12. Generation abbrechen → `generation_cancel`
13. Szenario > 24h alt öffnen → `scenario_revisit`
14. HR View erstmals öffnen → `feature_discovery` (hr_domain)
15. Multi-Year Generation starten → `feature_discovery` (multi_year)
16. Nächsten Tag besuchen → `return_visit`
17. Magic Link einlösen → `login_success` (server)
18. Generation abgeschlossen → `generation_complete` (server)
19. Checkout Stripe Session läuft ab → `checkout_abandon` (server)

**Dashboard:** `https://analytics.masem.at/dashboard/telling-cube`
