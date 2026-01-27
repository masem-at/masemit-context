# BuchungsvorgÃ¤nge masemIT e.U. â€“ FreeFinance

**Stand:** 24. JÃ¤nner 2026  
**Buchhaltung:** Einnahmen-Ausgaben-Rechnung (E/A)  
**USt-pflichtig:** Ja (freiwillig, Regelbesteuerung)  
**UVA-Intervall:** Quartal  
**Umstieg auf FreeFinance:** 01.10.2025

---

## 1. Zahlungsmittel

| Zahlungsmittel | Konto | Verwendung |
|----------------|-------|------------|
| GeschÃ¤ftskonto (easybank) | 2800 Bank | Normale GeschÃ¤ftszahlungen, Debitkarte |
| Stripe | 2790 Kreditkarte | Stripe-Einnahmen (Zwischenkonto) |
| Privat | 9600 Privat | Zahlungen vom Privatkonto |

---

## 2. RegelmÃ¤ÃŸige Ausgaben (Eingangsrechnungen)

### Bankspesen & GebÃ¼hren
| Beschreibung | Konto | USt | Bezahlt mit |
|--------------|-------|-----|-------------|
| KontofÃ¼hrungsgebÃ¼hren | 7790 | 0% | Bank 2800 |
| Manipulationsentgelt | 7790 | 0% | Bank 2800 |
| Kartenentgelt | 7790 | 0% | Bank 2800 |
| Mahnspesen | 7790 | 0% | Bank 2800 |
| Stripe-GebÃ¼hren | 7790 | 0% | Stripe 2790 |

### Hosting & Internet
| Beschreibung | Konto | USt | Bezahlt mit |
|--------------|-------|-----|-------------|
| easyname (Domains) | 7381 InternetgebÃ¼hren | 20% | Bank/Stripe |
| Vercel (Hosting) | 7381 InternetgebÃ¼hren | 20% (RC) | Stripe/Privat |
| Hosttech (E-Mail) | 7381 InternetgebÃ¼hren | 20% | Bank/Privat |

### Software & Lizenzen (SaaS)
| Beschreibung | Konto | USt | Bezahlt mit |
|--------------|-------|-----|-------------|
| ProSaldo.net | 7480 Lizenzaufwand | 20% | Bank 2800 |
| Canva | 7480 Lizenzaufwand | 20% (RC) | Bank/Privat |
| NeonDB | 7480 Lizenzaufwand | 20% (RC) | Stripe/Privat |
| Andere SaaS-Tools | 7480 Lizenzaufwand | 20% | variabel |

### Versicherung & Soziales
| Beschreibung | Konto | USt | Bezahlt mit |
|--------------|-------|-----|-------------|
| SVS-BeitrÃ¤ge | 6580 | 0% | Bank 2800 |
| Betriebsversicherung | 7700 | 0% | Bank 2800 |

### Sonstiges
| Beschreibung | Konto | USt | Bezahlt mit |
|--------------|-------|-----|-------------|
| WKO Kammerumlage | 7780 | 0% | Bank 2800 |
| BÃ¼romaterial | 7600 | 20% | variabel |
| Fachliteratur | 7630 | 10% | variabel |
| Weiterbildung | 7770 | 20% | variabel |

---

## 3. Einnahmen (Ausgangsrechnungen)

### tellingCube â€“ ProduktverkÃ¤ufe
| Produkt | Brutto | Netto | USt (20%) | Konto | Bezahlt auf |
|---------|--------|-------|-----------|-------|-------------|
| Small (alt) | â‚¬29,00 | â‚¬24,17 | â‚¬4,83 | 4000 | Stripe |
| Starter | â‚¬9,00 | â‚¬7,50 | â‚¬1,50 | 4000 | Stripe |
| Professional | â‚¬19,00 | â‚¬15,83 | â‚¬3,17 | 4000 | Stripe |
| Team | â‚¬49,00 | â‚¬40,83 | â‚¬8,17 | 4000 | Stripe |
| Lifetime Pro | â‚¬99,00 | â‚¬82,50 | â‚¬16,50 | 4000 | Stripe |
| Lifetime Team | â‚¬299,00 | â‚¬249,17 | â‚¬49,83 | 4000 | Stripe |

**Hinweis:** Stripe Tax ist aktiviert, USt ist im Preis inkludiert (EUR).

### ChainSights â€“ Governance Reports (geplant)
| Produkt | Preis | Konto | USt |
|---------|-------|-------|-----|
| Basic Report | â‚¬99 | 4000 | 20% |
| Advanced Report | â‚¬199 | 4000 | 20% |

### Reverse Charge (B2B EU/Drittland)
| Kundenstandort | Konto | USt |
|----------------|-------|-----|
| EU mit UID | 4111 Reverse Charge EU | 0% |
| Drittland (nicht EU) | 4210 Reverse Charge Drittland | 0% |

---

## 4. hoki.help â€“ Spendenplattform

### Eingehende Spenden (Durchlaufposten)
| Beschreibung | Konto | USt |
|--------------|-------|-----|
| Spende von Privatperson | 4999 Durchlaufposten Einnahmen | 0% |

### Stripe-GebÃ¼hren (bleiben bei dir)
| Beschreibung | Konto | USt |
|--------------|-------|-----|
| Stripe Fee auf Spende | 7790 Bankspesen | 0% |

### Weitergabe an HoKi NÃ–
| Beschreibung | Konto | USt |
|--------------|-------|-----|
| Ãœberweisung an Hospiz NÃ– | 7999 Durchlaufposten Ausgaben | 0% |

**EmpfÃ¤nger:** Landesverband Hospiz NiederÃ¶sterreich  
**Registrierungsnummer:** SO-18203  
**IBAN:** AT78 3225 0000 0070 7760

---

## 5. Spenden an HoKi NÃ– (aus BetriebsvermÃ¶gen)

### 3% vom Netto-Umsatz (tellingCube/ChainSights)
| Beschreibung | Konto | USt | Absetzbar |
|--------------|-------|-----|-----------|
| Spende an Hospiz NÃ– | 7692 Spenden begÃ¼nstigte Organisationen | 0% | Bis 10% vom Gewinn |

---

## 6. Umbuchungen (kein Aufwand/Ertrag)

### Stripe-Auszahlung auf Bankkonto
| Vorgang | Von | Nach | Konto-Buchung |
|---------|-----|------|---------------|
| Auszahlung | Stripe (2790) | Bank (2800) | Umbuchung |

**Wichtig:** Die Stripe-Auszahlung ist KEINE Einnahme! Das Geld wurde bereits beim Verkauf als Einnahme erfasst.

### Privateinlage
| Vorgang | Beschreibung | Konto |
|---------|--------------|-------|
| Einlage | Ãœberweisung von Privatkonto aufs GeschÃ¤ftskonto | 9600 Privat |

### Privatentnahme
| Vorgang | Beschreibung | Konto |
|---------|--------------|-------|
| Entnahme | Ãœberweisung vom GeschÃ¤ftskonto aufs Privatkonto | 9600 Privat |

---

## 7. Stripe-Buchungsablauf

### Option A: Einzelbuchungen (genau, mehr Aufwand)

**Szenario:** Kunde kauft tellingCube Lifetime Pro fÃ¼r â‚¬99

| Schritt | Beschreibung | Konto | Betrag | USt | Bezahlt auf/mit |
|---------|--------------|-------|--------|-----|-----------------|
| 1 | Verkauf erfassen | 4000 Einnahmen | â‚¬99,00 brutto | 20% | Stripe |
| 2 | Stripe-GebÃ¼hr | 7790 Bankspesen | â‚¬2,50 | 0% | Stripe |
| 3 | Auszahlung | Umbuchung 2790 â†’ 2800 | â‚¬96,50 | - | - |

### Option B: Sammelbuchungen (einfacher, fÃ¼r EPU okay)

Statt jede Transaktion einzeln zu buchen, kannst du monatlich oder quartalsweise Sammelbuchungen machen:

| Buchung | Konto | Beschreibung | USt |
|---------|-------|--------------|-----|
| 1 | 4000 | Summe aller tellingCube/ChainSights VerkÃ¤ufe | 20% |
| 2 | 4999 | Summe aller hoki.help Spenden | 0% |
| 3 | 7790 | Summe aller Stripe-GebÃ¼hren | 0% |
| 4 | Umbuchung | Auszahlung Stripe â†’ Bank | - |

**Wichtig bei Sammelbuchungen:**
- Belege (Stripe-Rechnungen) trotzdem aufbewahren
- In der Beschreibung "Sammelbuchung Q4/2025" o.Ã¤. vermerken
- Stripe-Export als Nachweis speichern

---

## 8. Konkret: Q4/2025 Stripe-Buchungen

### Ãœbersicht Transaktionen (Okt-Dez 2025)

**tellingCube VerkÃ¤ufe:**
| Produkt | Anzahl | Einzelpreis | Gesamt brutto |
|---------|--------|-------------|---------------|
| Supporter S (Founding) | 4x | â‚¬29,00 | â‚¬116,00 |
| Lifetime Pro | 1x | â‚¬99,00 | â‚¬99,00 |
| **Summe** | | | **â‚¬215,00** |

**hoki.help Spenden:**
| Betrag | Anzahl | Gesamt |
|--------|--------|--------|
| â‚¬5,00 | 3x | â‚¬15,00 |
| â‚¬10,00 | 2x | â‚¬20,00 |
| â‚¬25,00 | 1x | â‚¬25,00 |
| **Summe** | | **â‚¬60,00** |

**Stripe-GebÃ¼hren:** ~â‚¬6,78 (aus Stripe Reports exportieren)

**Auszahlung:** â‚¬208,22 (13.01.2026)

### Buchungen in FreeFinance

| Nr | Datum | Beschreibung | Konto | Betrag | USt | Bezahlt auf/mit |
|----|-------|--------------|-------|--------|-----|-----------------|
| 1 | 31.12.2025 | tellingCube VerkÃ¤ufe Q4/2025 | 4000 | â‚¬215,00 | 20% | Stripe |
| 2 | 31.12.2025 | hoki.help Spenden Q4/2025 | 4999 | â‚¬60,00 | 0% | Stripe |
| 3 | 31.12.2025 | Stripe GebÃ¼hren Q4/2025 | 7790 | â‚¬6,78 | 0% | Stripe |
| 4 | 13.01.2026 | Stripe Auszahlung | Umbuchung | â‚¬208,22 | - | 2790 â†’ 2800 |

### Kontrolle Stripe-Saldo

```
+ â‚¬215,00  (tellingCube)
+ â‚¬60,00   (hoki.help)
- â‚¬6,78    (GebÃ¼hren)
= â‚¬268,22  (Stripe-Saldo vor Auszahlung)

- â‚¬208,22  (Auszahlung)
= â‚¬60,00   (verbleibend fÃ¼r hoki.help Weitergabe)
```

### Noch zu buchen: Weitergabe an HoKi NÃ–

| Datum | Beschreibung | Konto | Betrag | Bezahlt mit |
|-------|--------------|-------|--------|-------------|
| [bei Ãœberweisung] | Spendenweitergabe hoki.help | 7999 | â‚¬60,00 | Bank 2800 |

---

## 9. KontenÃ¼bersicht (eingeblendet)

### Einnahmen
- 4000 Einnahmen (ErlÃ¶se) 20%
- 4111 Reverse Charge EU 0%
- 4210 Reverse Charge Drittland 0%
- 4999 Durchlaufposten Einnahmen 0%

### Ausgaben
- 6580 SVS-BeitrÃ¤ge
- 7021 Geringwertiges Wirtschaftsgut
- 7200 Instandhaltung
- 7380 Telefon
- 7381 InternetgebÃ¼hren
- 7400 Mietaufwand (optional)
- 7405 Arbeitszimmer (optional)
- 7480 Lizenzaufwand
- 7600 BÃ¼romaterial
- 7692 Spenden begÃ¼nstigte Organisationen
- 7700 Versicherung
- 7740 Steuerberatung
- 7770 Weiterbildung
- 7785 MitgliedsbeitrÃ¤ge
- 7790 Bankspesen
- 7850 Sonstige Aufwendungen
- 7999 Durchlaufposten Ausgaben

### Zahlungsmittel
- 2790 Kreditkarte (fÃ¼r Stripe)
- 2800 Bank

### Sonstiges
- 9600 Privat

---

## 10. Wichtige Fristen

| Was | Frist |
|-----|-------|
| UVA Q4/2025 | 15. Februar 2026 |
| UVA Q1/2026 | 15. Mai 2026 |
| Jahres-USt (U1) 2025 | 30. Juni 2026 |
| Einkommensteuer (E1+E1a) 2025 | 30. Juni 2026 |

---

## 11. Tipps fÃ¼r den Alltag

### Schnellzuordnung Eingangsrechnungen

| Wenn du siehst... | â†’ Konto | USt |
|-------------------|---------|-----|
| Stripe-GebÃ¼hr, easybank GebÃ¼hr, Manipulation | 7790 Bankspesen | 0% |
| Vercel, easyname, Hosttech, Netlify | 7381 InternetgebÃ¼hren | 20% (oder RC) |
| ProSaldo, Canva, NeonDB, GitHub, Figma | 7480 Lizenzaufwand | 20% (oder RC) |
| SVS-Beitrag | 6580 SVS | 0% |
| WKO, Kammerumlage | 7780 Kammerumlage | 0% |
| Mario Semper (Einlage) | 9600 Privat | - |

### Monatlicher Workflow

1. **Bankbewegungen:** finAPI importiert automatisch â†’ nur noch zuordnen
2. **Stripe:** Einmal im Monat Sammelbuchung (Einnahmen + GebÃ¼hren)
3. **hoki.help:** Monatliche Ãœberweisung an HoKi NÃ–
4. **Belege:** PDFs aus E-Mail in FreeFinance hochladen oder verlinken

### FreeFinance Buchungsautomat

In der **KontoauszugsÃ¼bersicht** gibt es einen "Buchungsautomat" â€“ der kann wiederkehrende Buchungen automatisch zuordnen:
- Regeln fÃ¼r Lieferanten anlegen (z.B. "ProSaldo" â†’ 7480)
- Spart Zeit bei wiederkehrenden Abbuchungen
- PrÃ¼fen ob im Paket enthalten oder Zusatzfunktion

### Reverse Charge (RC) bei Auslandsrechnungen

| Lieferant aus... | Was tun |
|------------------|---------|
| **Ã–sterreich** | Normal mit 20% USt |
| **EU (mit UID)** | Reverse Charge â€“ 0% auf Rechnung, du fÃ¼hrst USt selbst ab |
| **Drittland (USA, etc.)** | Reverse Charge â€“ 0% auf Rechnung, du fÃ¼hrst USt selbst ab |

Bei RC-Rechnungen in FreeFinance den entsprechenden Steuersatz wÃ¤hlen (z.B. "Reverse Charge EU").

---

## 12. Offene Punkte / To-Do

### Erledigt âœ…
- [x] FreeFinance Grundeinstellungen
- [x] Steuernummer eingetragen (23 104/7291)
- [x] Jahr 2025 + 2026 angelegt (Quartal-UVA)
- [x] Zahlungsmittel: Bank, Stripe, Privat
- [x] Bankanbindung (finAPI) ab 01.10.2025
- [x] FinanzOnline Webservice-Benutzer angelegt
- [x] Konten ein-/ausgeblendet
- [x] Stripe-Buchungslogik verstanden

### Noch offen
- [ ] Anfangsbestand Bank per 01.10.2025 eintragen
- [ ] Bankbewegungen Q4/2025 zuordnen (in KontoauszugsÃ¼bersicht)
- [ ] Stripe-Buchungen Q4/2025 erfassen (siehe Abschnitt 8)
- [ ] Stripe-GebÃ¼hren aus Reports exportieren (genauer Betrag)
- [ ] Ãœberweisung an HoKi NÃ– durchfÃ¼hren + buchen
- [ ] UVA Q4/2025 erstellen (Frist: 15.02.2026)
- [ ] 3%-Spendenmodell steuerlich klÃ¤ren (Spende vs. Marketing)