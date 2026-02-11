# Decision Log — kurzum.app

Decisions from team discussions, party mode sessions, and ad-hoc conversations. Prevents repeated discussions and provides context for future development.

| Datum | Entscheidung | Begründung | Kontext |
|-------|-------------|------------|---------|
| 2026-02-11 | Kein Cookie Consent Banner | masemIT Analytics ist cookiefrei (LP-NFR6). Cloudflare Turnstile setzt nur technisch notwendiges Cookie — kein Consent nötig. Kein Dienst auf der Landing Page erfordert Einwilligung. | Party Mode: Mary, John, Winston, Sally, Paige. Story 4-2-cookie-consent-banner gestrichen, Sprint-Status bereinigt, Nummerierung angepasst (4-2 = SEO, 4-3 = Analytics). |
| 2026-02-11 | Decision Log eingeführt | Diskussionsergebnisse aus Party Mode gingen verloren — Entscheidungen müssen persistiert werden. | Party Mode: Vorschlag von Paige. Datei: `docs/decisions.md`, referenziert in `project-context.md`. |
