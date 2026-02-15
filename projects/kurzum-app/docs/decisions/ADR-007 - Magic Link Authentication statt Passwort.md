# ADR-007: Magic Link Authentication statt Passwort

**Datum:** 2026-02-15
**Status:** Entschieden

## Kontext

kurzum.app benoetigt eine Authentifizierung fuer den geschuetzten App-Bereich (FFG-Anforderung: alle Features hinter Auth). Die Zielgruppe — Handwerker/Monteure auf Baustellen — nutzt die App primaer auf dem Smartphone, oft mit Arbeitshandschuhen oder schmutzigen Haenden. Passwort-basierte Authentifizierung ist fuer diese Zielgruppe problematisch:

- Passwort-Eingabe auf dem Smartphone auf der Baustelle ist umstaendlich
- Passwort-Reset-Flows erhoehen die Komplexitaet
- Passwort-Hashing, Salting und Brute-Force-Schutz muessen implementiert werden
- Passwort-Datenbankfelder erhoehen die DSGVO-Angriffsflaeche

## Entscheidung

**Magic Link Authentication via E-Mail** (Resend als Provider). Kein Passwort, kein OAuth.

### Implementierung

**Login-Flow (`app/api/auth/login/route.ts`):**
1. Nutzer gibt E-Mail-Adresse ein
2. Cloudflare Turnstile Bot-Schutz + Upstash Rate-Limiting (pro E-Mail)
3. `findOrCreateUser()` — Auto-Registrierung bei erstem Login
4. Magic Link Token generiert (UUID, 15 Min gueltig, `lib/auth/magic-link.ts`)
5. E-Mail via Resend (fire-and-forget, blockiert nicht den Response)

**Verifikation (`app/api/auth/verify/route.ts`):**
1. GET-Request mit Token als Query-Parameter
2. Atomisches Verify + Mark-as-Used (`UPDATE ... WHERE token = ? AND usedAt IS NULL`)
3. Session erstellt (7 Tage gueltig, httpOnly Cookie)
4. Redirect zu `/app`

**Session-Management (`lib/auth/session.ts`):**
- Sessions in DB gespeichert (nicht JWT)
- httpOnly, secure, sameSite=lax Cookie
- `getAuthContext()` liest Session + User + Company in einem Query
- 7 Tage Gueltigkeit

## Begruendung

| Kriterium | Magic Link | Passwort |
|-----------|-----------|----------|
| UX auf Baustelle | Kein Tippen noetig | Passwort auf Smartphone schwierig |
| Sicherheit | Token einmalig + zeitlich begrenzt | Brute-Force-Risiko, Password-Reuse |
| DSGVO | Kein Passwort-Hash gespeichert | Zusaetzliche personenbezogene Daten |
| Implementierung | Einfach (Resend + DB) | Komplex (Hashing, Reset, Policies) |
| Auto-Registrierung | Natuerlich integriert | Separater Signup-Flow noetig |
| FFG Pilot-Phase | Ideal fuer eingeladene Tester | Ueberengineered fuer Prototyp |

## Konsequenzen

### Positiv
- Kein Passwort-Speicherung — reduzierte DSGVO-Angriffsflaeche
- Auto-Registrierung ermoeglicht nahtloses Onboarding fuer Pilot-Tester
- TOCTOU-sichere Token-Verifikation (atomisches UPDATE)

### Negativ/Risiko
- Abhaengigkeit von E-Mail-Zustellung (Resend SLA)
- Nutzer muss Zugang zum E-Mail-Postfach auf dem Smartphone haben
- 15-Minuten-Token-Fenster kann auf Baustelle knapp sein

### Spaeter evaluieren
- SMS/WhatsApp als alternativer Magic-Link-Kanal (Pilot-Feedback)
- Passkey/WebAuthn als passwordless Upgrade (Sprint 3+)
