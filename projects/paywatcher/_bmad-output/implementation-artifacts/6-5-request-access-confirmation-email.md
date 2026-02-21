# Story 6.5 — Request Access: Bestätigungs-Email & Use Case Pflichtfeld

**Epic:** 6 — Admin Dashboard & Onboarding Improvements
**Priority:** Should-have (Phase 1)
**Effort:** 0.5 Tage

---

## Kontext

Aktuell bekommt der Kunde nach dem Request Access Formular keine Rückmeldung per Email. Er wartet im Dunkeln. Außerdem ist das Use Case Feld optional — ohne Use Case kann Mario nicht bewerten ob ein Tenant erstellt werden soll.

## Änderungen

### 1. Use Case Feld → Pflichtfeld

- `Use Case` Textarea im Request Access Formular wird `required`
- Placeholder anpassen: "Tell us about your project — e.g. what are you building, how will you accept USDC payments?"
- Validierung: Mindestens 20 Zeichen (verhindert "test" oder "asdf")
- Fehlermeldung: "Please describe your use case so we can set up the right plan for you."

### 2. Bestätigungs-Email an den Kunden

Beim erfolgreichen Form Submit werden **zwei Emails** gesendet (beide via Resend):

**Email A — An contact@masem.at (existiert bereits):**
Keine Änderung. Enthält Name, Email, Company, Use Case.

**Email B — An den Kunden (NEU):**

```
Subject: PayWatcher — We received your request

Hi {name},

Thanks for your interest in PayWatcher!

We've received your request and will review it within 24 hours. 
If approved, we'll send you your API credentials and help you 
get set up.

In the meantime, you can check out our API docs: 
https://paywatcher.dev/docs

Questions? Reply to this email.

— Mario Semper
Founder, masemIT
```

**Absender:** mario@paywatcher.dev (oder contact@paywatcher.dev — je nachdem was in Resend als Domain verifiziert ist)

### 3. Success State im Formular

Nach erfolgreichem Submit zeigt das Formular:
- Grüner Checkmark
- "Request submitted! Check your email for confirmation."
- Kein erneutes Absenden möglich (Button disabled oder Formular ausgeblendet)

---

## Akzeptanzkriterien

- [ ] Use Case ist Pflichtfeld mit Min. 20 Zeichen
- [ ] Kunde erhält Bestätigungs-Email innerhalb von Sekunden nach Submit
- [ ] Email enthält Name-Personalisierung
- [ ] Success State wird nach Submit angezeigt
- [ ] Bestehende Email an contact@masem.at funktioniert weiterhin
- [ ] Resend Sender-Domain ist korrekt konfiguriert

---

## Nicht in Scope

- Automatische Tenant-Erstellung (Phase 2)
- Automatische Party-Anlage als Lead (Phase 2)
- Onboarding-Email nach Tenant-Erstellung (manuell durch Mario)
