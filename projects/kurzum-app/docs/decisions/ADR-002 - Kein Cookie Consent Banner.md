# ADR-002: Kein Cookie Consent Banner

**Datum:** 2026-02-11
**Status:** Entschieden

**Kontext:** Evaluierung ob die Landing Page einen Cookie Consent Banner braucht.
             Drei Dienste im Einsatz: masemIT Analytics, Cloudflare Turnstile, keine weiteren Tracker.

**Entscheidung:** Kein Cookie Consent Banner implementieren. Story 4-2 (Cookie Consent) gestrichen.

**Begründung:**
- masemIT Analytics ist cookiefrei (LP-NFR6) — setzt keine Cookies, kein Tracking
- Cloudflare Turnstile setzt nur ein technisch notwendiges Cookie — kein Consent erforderlich (TTDSG §25 Abs. 2)
- Kein Dienst auf der Landing Page erfordert Einwilligung nach DSGVO/TTDSG

**Alternativen betrachtet:**
- Cookie Consent Banner mit Opt-in → unnötig, da keine consent-pflichtigen Cookies
- Cookie Consent Banner "zur Sicherheit" → verschlechtert UX ohne rechtliche Notwendigkeit

**Konsequenzen:**
- Sprint-Status bereinigt: Story 4-2 gestrichen, Nummerierung angepasst (4-2 = SEO, 4-3 = Analytics)
- Falls zukünftig consent-pflichtige Dienste hinzukommen, muss ein Banner nachgerüstet werden

**Beteiligte:** Party Mode — Mary, John, Winston, Sally, Paige
