# Go-to-Market Strategy: PayWatcher

**Project:** PayWatcher (paywatcher.dev)
**Version:** 1.0
**Date:** 2026-02-17
**Author:** Mario Semper (PO)
**Status:** Draft
**Referenzen:** Product Brief, Market Research, MMS Requirement v2, CR-PW-001

---

## 1. Executive Summary

PayWatcher ist die einzige reine Payment-Verification-API für Stablecoins. Kein Custody, kein Payment Processing, keine Checkout-Flows. Nur Verifizierung: "Sag mir welche Zahlung du erwartest, ich sage dir wenn sie angekommen ist."

Die GTM-Strategie folgt dem masemIT-Playbook: Founder-led, value-first, authentisch. Kein Paid Marketing im MVP. Stattdessen: Community-Arbeit, technische Credibility, und die ersten 5–10 Tenants durch persönliche Beziehungen.

**Kernzahlen für die ersten 6 Monate:**

| Metrik | Ziel |
|--------|------|
| Tenants (Free) | 5–8 |
| Tenants (Paid) | 2–3 |
| MRR | €87–€200 |
| Verified Payments | 100+ |
| Product Hunt Launch | Nach Tenant #5 |

---

## 2. Positioning

### Was PayWatcher IST

- Payment Verification Layer — zwischen Raw-Blockchain-Daten und Business-Logik
- Analogie: "Die Bank bestätigt dir, dass die Überweisung angekommen ist. PayWatcher macht das für USDC."
- "Stripe Radar für Stablecoin Payments" — Intelligence und Verifizierung ohne Processing

### Was PayWatcher NICHT IST

- Kein Payment Processor (kein Checkout, kein Currency Conversion)
- Kein Custody Provider (fasst Funds nie an)
- Kein Exchange oder On/Off Ramp
- Keine Raw-Blockchain-Infrastruktur (kein RPC, keine Webhooks)

### Differenzierung

| Wir | Alle anderen |
|-----|-------------|
| Flat-Fee ($0.05/Verification) | 0.5–1% pro Transaktion |
| Non-custodial | Custodial |
| 3 Zeilen Code | SDK + Checkout-Flow + Account Setup |
| Nur Verifizierung | Checkout + Custody + Settlement + Conversion |
| Du behältst 100% der Zahlung | Processor nimmt 0.5–2% |

### Killer-Beispiel (in jeder Kommunikation verwenden)

> Dein Kunde zahlt $10.000 in USDC.
> Mit Coinbase Commerce zahlst du $100 Gebühr.
> Mit PayWatcher zahlst du $0.05.
> Gleiche Bestätigung. 2000x günstiger.

---

## 3. Target Segments (Priorität)

### Segment 1: Web3-Native Developers (Primary)

**Wer:** Indie-Devs und kleine Teams, die DApps/Tools bauen und USDC-Zahlungen akzeptieren wollen, ohne einen Payment Processor zu integrieren.

**Wo:** Discord (Alchemy, Base, Ethereum), Twitter/X (#BuildOnBase), Farcaster, Hackathons

**Pain:** "Ich will nicht Coinbase Commerce integrieren nur damit jemand mir USDC schicken kann. Und ich will nicht 1% abgeben."

**Hook:** Code-Beispiel. 3 Zeilen curl → Payment verifiziert. Kein SDK, kein Account, kein Checkout-Widget.

**Persona: Alex** — 28, Full-Stack Dev, baut ein SaaS Tool für DAOs, braucht USDC-Payment-Verification, hasst Overhead.

### Segment 2: Freelancer & Service Providers (Secondary)

**Wer:** Freelancer die Krypto-Zahlungen akzeptieren, aber kein ganzes Payment-System brauchen. Oft über Plattformen wie Opolis, Request Network, oder Superfluid abgerechnet.

**Wo:** LinkedIn, Freelancer-Communities, Web3 Job Boards (Crypto Jobs List, Web3 Career)

**Pain:** "Mein Kunde hat mir USDC geschickt. Ich muss manuell in Basescan nachschauen ob es angekommen ist."

**Hook:** "Automatische Bestätigung wenn die Zahlung ankommt. Webhook an dein System. Fertig."

**Persona: Sandra** — 35, UX Freelancerin, akzeptiert Krypto, will nicht manuell Transaktionen tracken.

### Segment 3: B2B Stablecoin Transfers (Langfristig)

**Wer:** Unternehmen die High-Value B2B-Zahlungen in Stablecoins abwickeln und dafür keine 1% an einen Processor zahlen wollen.

**Wo:** LinkedIn, Stablecoin-fokussierte Events, CFO/Treasury Communities

**Pain:** "Wir zahlen $500 pro $50K-Transaktion an Coinbase Commerce. Das ist absurd für eine simple Transferbestätigung."

**Hook:** Das $10.000-Beispiel. Flat-Fee vs. Percentage.

---

## 4. Launch-Phasen

### Phase 0: Pre-Launch (jetzt – bis Frontend fertig)

**Ziel:** Boden bereiten, erste Conversations starten.

| Aktion | Kanal | Aufwand | Wer | Status |
|--------|-------|---------|-----|--------|
| "Building in Public" Post #1 | LinkedIn + Farcaster | 1h | Mario | ✅ |
| — Thema: "Warum gibt es keine reine Verification-API für Stablecoins?" | | | |
| — Keine Produkt-Erwähnung, nur die Frage stellen | | | |
| — Cross-Post: Bluesky, dev.to (zero extra effort) | | | |
| Landing Page (paywatcher.dev) mit "Request Access" live | Web | Teil des Frontend-Projekts | Oliver + BMAD |
| API Docs Draft auf docs.paywatcher.dev | Web | Teil des Frontend-Projekts | Oliver + BMAD |
| Base/Alchemy Discord beitreten, anfangen zu helfen | Discord | 15 min/Tag | Mario |
| ChainSights-Newsletter Teaser ("Wir bauen was Neues") | Email | 30 min | Mario |
| Mirror-Profil einrichten, erster "Why we're building this" Post | Mirror | 2h einmalig | Mario |

**KPI:** 3+ Conversations über Stablecoin Payment Pain Points.

### Phase 1: Soft Launch (Frontend live – Woche 1–4)

**Ziel:** Erste 3 Tenants onboarden. Lernen.

| Aktion | Kanal | Aufwand | Wer |
|--------|-------|---------|-----|
| "Building in Public" Post #2 | LinkedIn + Farcaster | 1h | Mario |
| — Thema: "3 Zeilen Code, Payment verifiziert. Hier ist wie." | | | |
| — Code-Snippet als Bild, Link zu Docs | | | |
| — Cross-Post: Mirror (Long-Form), Hashnode (Tutorial-Format) | | | |
| Persönliche Outreach an 5–10 potenzielle Tenants | Email / Discord DM | 2h/Woche | Mario |
| — Aus ChainSights DAO-Community-Kontakten | | | |
| — Aus Base/Alchemy Discord Kontakten | | | |
| Erste Request-Access-Anfragen bearbeiten (<24h) | Email | je 30 min | Mario |
| Tenant-Onboarding: Key + Secret übergeben, Quick Call | Zoom/Discord | je 30 min | Mario |
| Feedback sammeln, Dashboard iterieren | Gespräch | laufend | Mario |

**KPI:** 3 aktive Tenants, 1 davon macht echte Payments.

### Phase 2: Community Launch (Woche 5–8)

**Ziel:** Von 3 auf 5–8 Tenants, erste Paid Conversion.

| Aktion | Kanal | Aufwand | Wer |
|--------|-------|---------|-----|
| "Building in Public" Post #3–5 | LinkedIn + Farcaster | 1h/Woche | Mario |
| — Thema: Learnings, echte Zahlen, Tenant-Feedback | | | |
| — "Wir haben $X in Stablecoin Payments verifiziert" | | | |
| — Long-Form Version auf Mirror (monatlich) | | | |
| Dev-Tutorial: "Accept USDC Payments in 10 Minutes" | Hashnode + Blog (paywatcher.dev/blog) | 4h einmalig | Mario |
| — Cross-Post: dev.to (automatisch von Hashnode) | | | |
| Forum-Posts mit Mehrwert | Base Discord, Ethereum Magicians | 30 min/Woche | Mario |
| — Nicht "schau mein Produkt an", sondern "hier ist wie man X löst" | | | |
| Pricing Page live schalten | Web | Teil des Frontend | Oliver |
| Erster Paid-Tier-Tenant onboarden | Persönlich | 1h | Mario |

**KPI:** 5+ Tenants (davon 1–2 Paid), 50+ Verified Payments, Blog Post >200 Views.

### Phase 3: Product Hunt + Skalierung (Woche 9–12)

**Ziel:** Product Hunt Launch, 10 Tenants.

**Voraussetzung:** Mindestens 5 aktive Tenants mit echten Payments und mindestens 1 Testimonial.

| Aktion | Kanal | Aufwand | Wer |
|--------|-------|---------|-----|
| Product Hunt Launch vorbereiten | PH | 4h Prep + Launch Day | Mario |
| — "Hunter" mit Reichweite finden (aus ChainSights-Netz) | | | |
| — Landing Page auf PH-Traffic optimieren | | | |
| — Launch-Day Community mobilisieren (Farcaster, LinkedIn, Discord) | | | |
| Hacker News "Show HN" Post | HN | 2h | Mario |
| — Technischer Fokus: Architektur, Unique-Amount-Trick, Non-Custodial | | | |
| Mirror Post: "How we built PayWatcher" (technischer Deep Dive) | Mirror | 3h | Mario |
| — Cross-Post: Hashnode, Farcaster Thread, LinkedIn | | | |
| Self-Service Signup evaluieren (Phase 2 Entscheidung) | Intern | — | Mario |

**KPI:** PH Top 10 des Tages, 10+ Tenants, €87+ MRR.

---

## 5. Content-Strategie

### Content-Typen (Priorität)

| Typ | Beispiel | Kanal | Frequenz |
|-----|---------|-------|----------|
| **Code-first Posts** | curl-Beispiel, SDK Snippet, "3 Lines to Verify" | Farcaster, LinkedIn | 2x/Woche |
| **Building in Public** | Entscheidungen, Zahlen, Learnings | LinkedIn, Farcaster | 2x/Woche |
| **Long-Form / Deep Dives** | "Why verification-only?", "How we built PayWatcher" | Mirror | 1–2x/Monat |
| **Tutorials** | "Accept USDC in 10 Min", "Webhooks with PayWatcher" | Hashnode → Cross-Post dev.to + Blog | 1x/Monat |
| **Comparison** | "PayWatcher vs. Coinbase Commerce: Cost Breakdown" | Hashnode + Blog | 1x/Monat |
| **Forum-Hilfe** | Fragen zu Stablecoin-Payments beantworten, PayWatcher als eine Option erwähnen | Discord, Farcaster | laufend |

### Content Rules (masemIT-Playbook)

1. **Value first, product second.** Jeder Post muss auch ohne PayWatcher nützlich sein.
2. **Code > Words.** Wenn du es als Code-Snippet zeigen kannst, zeig es als Code-Snippet.
3. **Keine Superlative.** Nicht "the best" oder "revolutionary". Stattdessen: Zahlen, Vergleiche, konkretes Beispiel.
4. **Ehrlich über Limitationen.** "Aktuell nur USDC auf Base. Ethereum und Polygon kommen in Phase 2."
5. **Deutsch auf LinkedIn, Englisch überall sonst.** (Wie bei ChainSights.) Mirror, Farcaster, Hashnode, dev.to = immer Englisch.

### Der "Killer-Post" (vorbereiten für Launch Day)

**Format:** Mirror Deep Dive + LinkedIn Post + Farcaster Thread + HN Show HN

**Struktur:**
1. Das Problem: "Es gibt keine reine Verification-API für Stablecoins."
2. Die Optionen: Payment Processor (1% Fee) oder DIY (Wochen Entwicklung).
3. Der Vergleich: $10K Zahlung → $100 bei Coinbase vs. $0.05 bei PayWatcher.
4. Das Code-Beispiel: 3 Zeilen curl.
5. CTA: "Wir sind live. Request Access: paywatcher.dev"

---

## 6. Channel-Strategie

### Tier 1 — Aktiv bespielen (eigener Content)

| Kanal | Zweck | Frequenz | Sprache |
|-------|-------|----------|---------|
| **LinkedIn** (Mario persönlich) | Building in Public, B2B-Segment, DACH-Audience | 2x/Woche | DE + EN |
| **Farcaster** | Code-Posts, kurze Takes, Web3-Native Developer Community | 2x/Woche | EN |
| **Mirror** | Long-Form: "Why we built this", technische Deep Dives, Monatsupdates | 1–2x/Monat | EN |
| **paywatcher.dev/blog** | Tutorials, Comparisons, API Docs | 1x/Monat | EN |

**Warum diese 4:** LinkedIn hat die Reach und den B2B-Kanal. Farcaster ist wo die Base/Ethereum Devs sind. Mirror gibt Web3-Credibility (on-chain publishing, ENS-Domain). Der eigene Blog ist für SEO und als permanente Referenz.

### Tier 2 — Cross-Post (zero extra effort)

| Kanal | Quelle | Mechanismus |
|-------|--------|-------------|
| **Hashnode** | Tutorials vom Blog | Hashnode-Canonical-URL auf eigenen Blog setzen, SEO-Boost mitnehmen |
| **dev.to** | Von Hashnode | Hashnode hat built-in dev.to Cross-Post, ein Toggle |
| **Bluesky** | Farcaster-Posts | Copy-Paste, dauert 30 Sekunden pro Post |

**Regel:** Kein extra Content für Tier 2. Alles was dort erscheint, existiert bereits in Tier 1. Wenn Cross-Post mehr als 2 Minuten dauert, lassen.

### Tier 3 — Ignorieren

| Kanal | Warum nicht |
|-------|-------------|
| **Threads** | Meta-Ecosystem, kaum Developer-Audience, kein Web3-Fokus |
| **Mastodon** | Zu fragmentiert, zu wenig Reach, Web3-Devs sind dort nicht |
| **X/Twitter** | Account gesperrt. Kein Zugang. |

### Community Channels (nicht owned, reinhelfen)

| Kanal | Taktik | Priorität |
|-------|--------|-----------|
| **Base Discord** | Helfen, nicht promoten. Bei Payment-Fragen PayWatcher als Option erwähnen. | Hoch |
| **Alchemy Discord** | Gleich wie Base. | Hoch |
| **Farcaster /base Channel** | Code-Posts, kurze Takes zum Thema Stablecoin Payments | Hoch |
| **Ethereum Magicians / Research Forum** | Technische Diskussionen, Payment-Standards | Niedrig |

### Paid Channels

**Keine im MVP.** Erst evaluieren wenn Organic traction bewiesen ist (>10 Tenants).

### Content-Flow-Diagramm

```
Mario schreibt Content
         ↓
    ┌────┴────────────────────────┐
    ↓                             ↓
Short-Form                    Long-Form
(Code, Takes, Updates)        (Deep Dives, Tutorials)
    ↓                             ↓
┌───┴───┐                   ┌────┴────┐
↓       ↓                   ↓         ↓
LinkedIn  Farcaster         Mirror    Hashnode
            ↓                           ↓
         Bluesky                      dev.to
       (Cross-Post)               (Cross-Post)
                                      ↓
                              paywatcher.dev/blog
                                (Canonical URL)
```

---

## 7. Pricing-Kommunikation

### Pricing (bestätigt im Product Brief)

| Tier | Preis | Verifications/Monat | Zielgruppe |
|------|-------|---------------------|-----------|
| **Free** | $0 | 50 | Dev/Testing |
| **Starter** | $29 | 500 | Kleine Merchants |
| **Pro** | $99 | 2.000 | Growing Businesses |
| **Enterprise** | Custom | Unlimited | High-Volume |

### Pricing-Narrativ

**Don't say:** "Wir sind günstiger."
**Do say:** "Wir haben ein fundamental anderes Modell."

Der Punkt ist nicht, dass PayWatcher billiger ist. Der Punkt ist, dass Percentage-based Pricing für Verification keinen Sinn macht. Eine Bank verlangt auch nicht 1% um dir zu bestätigen, dass eine Überweisung angekommen ist.

### Vergleichstabelle (für Landing Page + Blog)

| | Coinbase Commerce | NOWPayments | PayWatcher |
|---|---|---|---|
| $100 Payment | $1.00 | $0.50 | $0.05 |
| $1,000 Payment | $10.00 | $5.00 | $0.05 |
| $10,000 Payment | $100.00 | $50.00 | $0.05 |
| $100,000 Payment | $1,000.00 | $500.00 | $0.05 |
| Custodial? | Ja | Ja | Nein |
| Checkout UI nötig? | Ja | Ja | Nein |

---

## 8. Managed MVP — Onboarding-Flow

Da das MVP managed ist (kein Self-Service Signup), ist der Onboarding-Flow persönlich:

### Tenant-Acquisition-Flow

```
1. Tenant entdeckt PayWatcher (Content, Discord, Empfehlung)
         ↓
2. Request Access (Formular auf paywatcher.dev)
         ↓
3. Mario reviewt (<24h)
         ↓
4. Kurzer Call oder Chat (Use Case verstehen, Expectations setzen)
         ↓
5. Mario legt Tenant an (Admin Dashboard: POST /v1/admin/paywatcher/tenants)
         ↓
6. API Key + Webhook Secret per sicherem Kanal übermitteln (Signal, verschlüsselte Email)
         ↓
7. Tenant integriert (Docs + Code Examples als Referenz)
         ↓
8. Mario monitort erste Payments (Admin Health Dashboard)
         ↓
9. Feedback-Loop (nach 1 Woche, nach 1 Monat)
```

### Was Mario bei jedem Onboarding lernt

- Welcher Use Case? (DApp, Freelancer, B2B, E-Commerce?)
- Wie haben sie PayWatcher gefunden?
- Was war die Alternative? (DIY, Processor, gar nichts?)
- Was fehlt ihnen? (Feature Requests → Phase 2 Input)

---

## 9. Cross-Promotion mit masemIT-Produkten

| Produkt | Cross-Promotion |
|---------|----------------|
| **ChainSights** | "Von den Machern von ChainSights" als Trust Signal. Newsletter-Teaser. Shared Web3-Community-Kontakte. Farcaster-Audience überschneidet sich. |
| **hoki.help** | PayWatcher akzeptiert Spenden in USDC via PayWatcher selbst (Dogfooding). 3% Revenue Donation. |
| **masem.at** | Footer-Link, "Powered by masemIT"-Badge. |
| **Mirror (chainsightsone.eth)** | PayWatcher-Posts unter bestehendem ENS-Domain publizieren → sofort Web3-Credibility. |

---

## 10. Success Metrics & Meilensteine

### Monat 1–2 (Soft Launch)

| Metrik | Ziel |
|--------|------|
| Request Access Submissions | 10+ |
| Onboarded Tenants | 3 |
| Verified Payments | 10+ |
| Blog Post publiziert | 1 |
| Discord/Community Interactions | 20+ |

### Monat 3–4 (Community Launch)

| Metrik | Ziel |
|--------|------|
| Onboarded Tenants | 5–8 |
| Paid Tenants | 1–2 |
| Verified Payments | 50+ |
| MRR | $29–$58 |
| Blog Posts | 2 |
| Testimonial/Quote | 1 |

### Monat 5–6 (Product Hunt + Skalierung)

| Metrik | Ziel |
|--------|------|
| Onboarded Tenants | 10+ |
| Paid Tenants | 2–3 |
| Verified Payments | 100+ |
| MRR | $87–$200 |
| Product Hunt Launch | Top 10 am Launch Day |
| Self-Service Decision | Ja/Nein für Phase 2 |

---

## 11. Risiken & Mitigations

| Risiko | Wahrscheinlichkeit | Mitigation |
|--------|---------------------|------------|
| Zu wenig Request Access Anfragen | Mittel | Content-Frequenz erhöhen, direktere Outreach, Base/Alchemy Discord intensiver |
| Tenants onboarden, nutzen es aber nicht | Hoch | Proaktiver Follow-up nach 3 Tagen, Onboarding-Call mit konkretem Integration-Plan |
| Coinbase Commerce wird günstiger | Niedrig | Positioning ist nicht nur Preis — es ist Non-Custodial, kein Checkout, Developer-First. Preis ist ein Argument, nicht DAS Argument. |
| Base Network Problem (Downtime, Reorg) | Niedrig | Monitoring, Admin Health Dashboard, transparente Kommunikation an Tenants |
| Regulatory Bedenken ("Ist das ein Money Service Business?") | Mittel | Klare Kommunikation: PayWatcher ist Verification, nicht Processing. Nie Custody. Juristisch absichern wenn >€10K MRR. |

---

## 12. Budget

| Posten | Kosten |
|--------|--------|
| Domain (paywatcher.dev) | $10.81/Jahr |
| Vercel Pro | $0 (bestehend) |
| NeonDB | $0 (bestehend) |
| Alchemy RPC | $0 (Free Tier) |
| QStash | ~$3–5/Monat |
| Ledger Nano S Plus | €49 (einmalig) |
| Product Hunt Launch | $0 |
| Paid Ads | $0 |
| **Total Jahr 1** | **~$120** |

---

## 13. Timeline-Übersicht

```
        Feb 2026         März 2026          Apr 2026           Mai 2026
        ─────────        ──────────         ──────────         ──────────
Backend │████████████│                  │                  │
Endpoints│  Stories 0-3│                  │                  │
        │            │                  │                  │
Frontend│      ████████████████████│                  │                  │
        │      Oliver + BMAD      │                  │                  │
        │                         │                  │                  │
Pre-     │██████│                  │                  │                  │
Launch   │BIP#1 │                  │                  │                  │
        │Discord│                  │                  │                  │
        │       │                  │                  │                  │
Soft     │       │████████████████│                  │                  │
Launch   │       │ Request Access │                  │                  │
        │       │ First 3 Tenants│                  │                  │
        │       │                │                  │                  │
Community│       │                │████████████████│                  │
Launch   │       │                │ Blog, Forums   │                  │
        │       │                │ 5-8 Tenants    │                  │
        │       │                │                │                  │
PH       │       │                │                │████████│         │
Launch   │       │                │                │ PH + HN│         │
        │       │                │                │10+ Ten.│         │
```

---

*Nächster Schritt: Team-Review, dann in die Content-Pipeline (erster "Building in Public" Post drafting).*
