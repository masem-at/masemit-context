# Web3, DAOs & On-Chain Analytics — Eine Einführung

**Autor:** Mario (masemIT e.U.)  
**Datum:** Dezember 2025  
**Zweck:** Grundlagenwissen für den Einstieg in die Web3-Analytics-Branche

---

## Inhaltsverzeichnis

1. [Was ist Web3?](#1-was-ist-web3)
2. [Blockchain-Grundlagen](#2-blockchain-grundlagen)
3. [Smart Contracts](#3-smart-contracts)
4. [Wallets — Die digitale Identität](#4-wallets--die-digitale-identität)
5. [Tokens — Die Bausteine von Web3](#5-tokens--die-bausteine-von-web3)
6. [DAOs — Dezentrale Autonome Organisationen](#6-daos--dezentrale-autonome-organisationen)
7. [Governance in Web3](#7-governance-in-web3)
8. [Das Problem: Wallets lügen](#8-das-problem-wallets-lügen)
9. [On-Chain Analytics](#9-on-chain-analytics)
10. [Die wichtigsten Akteure im Markt](#10-die-wichtigsten-akteure-im-markt)
11. [DeFi — Dezentrale Finanzen](#11-defi--dezentrale-finanzen)
12. [NFTs — Non-Fungible Tokens](#12-nfts--non-fungible-tokens)
13. [Regulierung: MiCA und die EU](#13-regulierung-mica-und-die-eu)
14. [Technische Infrastruktur](#14-technische-infrastruktur)
15. [Glossar](#15-glossar)

---

## 1. Was ist Web3?

### Die Evolution des Internets

| Generation | Zeitraum | Charakteristik | Beispiele |
|------------|----------|----------------|-----------|
| **Web1** | 1990-2004 | Read-only, statische Seiten | Geocities, frühe Websites |
| **Web2** | 2004-heute | Read-write, soziale Plattformen | Facebook, YouTube, Twitter |
| **Web3** | 2015-heute | Read-write-own, dezentral | Ethereum, DAOs, DeFi |

### Web3 in einem Satz

> **Web3 ist die Vision eines Internets, in dem Nutzer ihre Daten, ihre Identität und ihre digitalen Werte selbst besitzen — ohne zentrale Plattformen als Vermittler.**

### Die Kernprinzipien von Web3

**1. Dezentralisierung**
- Keine einzelne Instanz kontrolliert das Netzwerk
- Daten sind auf tausende Computer verteilt
- Entscheidungen werden kollektiv getroffen

**2. Ownership (Eigentum)**
- Nutzer besitzen ihre digitalen Assets wirklich
- Tokens, NFTs, Coins gehören dir — nicht einer Plattform
- "Not your keys, not your coins"

**3. Permissionless (Erlaubnisfrei)**
- Jeder kann teilnehmen
- Keine Gatekeeper, keine Genehmigungen
- Open Source, offen für alle

**4. Trustless (Vertrauenslos)**
- Du musst niemandem vertrauen
- Code (Smart Contracts) führt Regeln automatisch aus
- Mathematik statt Vertrauen

### Warum ist Web3 relevant?

Für traditionelle Unternehmen und Analysten:

- **Neue Geschäftsmodelle**: DAOs, Token-basierte Anreize
- **Transparenz**: Alle Transaktionen sind öffentlich einsehbar
- **Globale Märkte**: 24/7, ohne Grenzen
- **Regulatorischer Druck**: MiCA in der EU macht Web3-Compliance zum Thema

---

## 2. Blockchain-Grundlagen

### Was ist eine Blockchain?

> **Eine Blockchain ist ein dezentrales, unveränderliches Hauptbuch (Ledger), das Transaktionen chronologisch und transparent aufzeichnet.**

Stell dir vor:
- Ein Google Spreadsheet, das jeder lesen kann
- Aber niemand kann alte Einträge ändern
- Und tausende Computer haben eine Kopie

### Wie funktioniert es?

```
Block 1          Block 2          Block 3
┌──────────┐     ┌──────────┐     ┌──────────┐
│ Tx 1     │     │ Tx 4     │     │ Tx 7     │
│ Tx 2     │ ──► │ Tx 5     │ ──► │ Tx 8     │
│ Tx 3     │     │ Tx 6     │     │ Tx 9     │
│ Hash: A1 │     │ Hash: B2 │     │ Hash: C3 │
│ Prev: 00 │     │ Prev: A1 │     │ Prev: B2 │
└──────────┘     └──────────┘     └──────────┘
```

**Jeder Block enthält:**
- Transaktionen (Tx)
- Einen eigenen Hash (digitaler Fingerabdruck)
- Den Hash des vorherigen Blocks

**Warum ist das sicher?**
- Ändert man Block 1, ändert sich sein Hash
- Dann passt der "Prev"-Verweis in Block 2 nicht mehr
- Man müsste alle folgenden Blöcke neu berechnen
- Bei tausenden Computern ist das praktisch unmöglich

### Die wichtigsten Blockchains

| Blockchain | Gegründet | Stärke | Schwäche |
|------------|-----------|--------|----------|
| **Bitcoin** | 2009 | Erste, sicherste, "digitales Gold" | Langsam, keine Smart Contracts |
| **Ethereum** | 2015 | Smart Contracts, DeFi, DAOs | Hohe Gebühren (Gas) |
| **Polygon** | 2017 | Schnell, günstig, Ethereum-kompatibel | Weniger dezentral |
| **Solana** | 2020 | Sehr schnell, günstig | Zentralisierter, Ausfälle |
| **Arbitrum** | 2021 | Ethereum Layer 2, günstig | Abhängig von Ethereum |

### Für ChainSights relevant

- **Ethereum** ist der wichtigste Markt für DAOs und Governance
- Die meisten Governance-Tokens sind auf Ethereum
- **Layer 2s** (Polygon, Arbitrum) werden wichtiger
- Multi-Chain-Support wird später relevant

---

## 3. Smart Contracts

### Was ist ein Smart Contract?

> **Ein Smart Contract ist ein Programm, das auf der Blockchain läuft und automatisch ausgeführt wird, wenn bestimmte Bedingungen erfüllt sind.**

### Analogie: Der Getränkeautomat

```
WENN: Du wirfst €2 ein
UND: Du drückst Knopf B3
DANN: Automat gibt Cola aus

Kein Vertrauen nötig. Kein Mitarbeiter nötig.
Der Mechanismus (Code) garantiert das Ergebnis.
```

### Smart Contract Beispiel (vereinfacht)

```solidity
// Governance-Abstimmung
contract Voting {
    mapping(address => bool) public hasVoted;
    uint public yesVotes;
    uint public noVotes;
    
    function vote(bool _yes) public {
        require(!hasVoted[msg.sender], "Already voted");
        hasVoted[msg.sender] = true;
        
        if (_yes) {
            yesVotes++;
        } else {
            noVotes++;
        }
    }
}
```

**Was passiert hier:**
- Jede Wallet-Adresse kann einmal abstimmen
- Der Contract zählt automatisch
- Niemand kann die Regeln ändern
- Alles ist transparent und überprüfbar

### Warum sind Smart Contracts wichtig für ChainSights?

- **Governance-Abstimmungen** laufen über Smart Contracts
- Alle Votes sind **on-chain** und auswertbar
- Wir können sehen: Welche Wallet hat wann wie abgestimmt
- Aber: Wir wissen nicht, ob 10 Wallets = 10 Menschen

---

## 4. Wallets — Die digitale Identität

### Was ist eine Wallet?

> **Eine Wallet ist deine digitale Identität in Web3 — ein Schlüsselpaar, das dir erlaubt, Transaktionen zu signieren und Assets zu besitzen.**

### Die zwei Schlüssel

| Schlüssel | Funktion | Analogie |
|-----------|----------|----------|
| **Public Key** (Adresse) | Deine "Kontonummer", öffentlich teilbar | IBAN |
| **Private Key** | Dein geheimer Zugangscode, NIE teilen | PIN + TAN |

**Beispiel einer Ethereum-Adresse:**
```
0x742d35Cc6634C0532925a3b844Bc9e7595f8B2E1
```

### Wallet-Typen

| Typ | Beschreibung | Beispiele |
|-----|--------------|-----------|
| **Hot Wallet** | Online, bequem, weniger sicher | MetaMask, Rainbow |
| **Cold Wallet** | Offline, sicherer, umständlicher | Ledger, Trezor |
| **Custodial** | Dritter verwahrt Schlüssel | Coinbase, Binance |
| **Non-custodial** | Du verwahrst Schlüssel selbst | MetaMask, Ledger |

### Das Problem mit Wallets (für ChainSights)

**Ein Mensch kann beliebig viele Wallets haben:**

```
Person "Elena":
├── Wallet 1: 0x742d... (Haupt-Wallet)
├── Wallet 2: 0x8f3a... (für DeFi)
├── Wallet 3: 0x1bc9... (für NFTs)
└── Wallet 4: 0x5e2f... (Airdrop-Farming)
```

**Wenn Elena 4x abstimmt, sieht es aus wie 4 Personen.**

Das ist das Kernproblem, das ChainSights löst.

### Wallet-Clustering (Was ChainSights tut)

**Hinweise, dass Wallets zusammengehören:**

| Signal | Beschreibung |
|--------|--------------|
| **Funding-Quelle** | Alle Wallets wurden von derselben Adresse finanziert |
| **Timing** | Transaktionen innerhalb von Sekunden |
| **Interaktionsmuster** | Dieselben DApps, dieselben Tokens |
| **Gas-Zahler** | Eine Wallet zahlt Gas für andere |

---

## 5. Tokens — Die Bausteine von Web3

### Was sind Tokens?

> **Tokens sind programmierbare digitale Einheiten auf einer Blockchain — sie können Währung, Stimmrechte, Eigentumsanteile oder alles andere repräsentieren.**

### Token-Standards (Ethereum)

| Standard | Typ | Beschreibung | Beispiele |
|----------|-----|--------------|-----------|
| **ERC-20** | Fungible | Austauschbar, wie Geld | USDC, UNI, AAVE |
| **ERC-721** | Non-Fungible (NFT) | Einzigartig, nicht austauschbar | Bored Apes, CryptoPunks |
| **ERC-1155** | Multi-Token | Kombiniert fungible + NFT | Gaming-Items |

### Governance Tokens

**Besonders relevant für ChainSights:**

| Token | DAO | Funktion |
|-------|-----|----------|
| **UNI** | Uniswap | Stimmrecht über Protokoll-Änderungen |
| **AAVE** | Aave | Governance + Sicherheits-Modul |
| **ENS** | ENS DAO | Stimmrecht über ENS-Entwicklung |
| **ARB** | Arbitrum | Layer-2-Governance |

**Wie Governance-Tokens funktionieren:**
- 1 Token = 1 Stimme (meistens)
- Mehr Tokens = mehr Einfluss
- Problem: "Whales" (Großhalter) dominieren

---

## 6. DAOs — Dezentrale Autonome Organisationen

### Was ist eine DAO?

> **Eine DAO ist eine Organisation, die durch Smart Contracts gesteuert wird und deren Regeln transparent und unveränderlich auf der Blockchain gespeichert sind.**

### DAO vs. Traditionelle Organisation

| Aspekt | Traditionelle Firma | DAO |
|--------|--------------------|----|
| **Struktur** | Hierarchisch | Flach, Token-basiert |
| **Entscheidungen** | Management/Vorstand | Token-Holder-Abstimmung |
| **Transparenz** | Begrenzt (intern) | Vollständig (on-chain) |
| **Mitgliedschaft** | Verträge, Anstellung | Token-Besitz |
| **Geografie** | Lokal/national | Global, grenzenlos |
| **Rechtsform** | GmbH, AG, etc. | Oft unklar |

### Arten von DAOs

| DAO-Typ | Zweck | Beispiele |
|---------|-------|-----------|
| **Protocol DAO** | Governance eines DeFi-Protokolls | Uniswap, Aave, Compound |
| **Investment DAO** | Gemeinsame Investments | The LAO, MetaCartel Ventures |
| **Service DAO** | Dienstleistungen anbieten | Raid Guild, LexDAO |
| **Social DAO** | Community, Kultur | Friends With Benefits |
| **Collector DAO** | NFTs sammeln | PleasrDAO, Flamingo |
| **Media DAO** | Content, Journalismus | Bankless DAO |

### DAO-Statistiken (2025)

- **16.000+** DAOs weltweit
- **$25+ Milliarden** in DAO-Treasuries
- **Millionen** von Token-Holdern

### Wie eine DAO funktioniert

```
┌─────────────────────────────────────────────────────────┐
│                     DAO-Lifecycle                        │
└─────────────────────────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────┐
│  1. PROPOSAL (Vorschlag)                                 │
│     - Jemand erstellt einen Vorschlag                   │
│     - Z.B. "Lasst uns 100k für Marketing ausgeben"      │
└─────────────────────────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────┐
│  2. DISCUSSION (Diskussion)                              │
│     - Community diskutiert im Forum/Discord              │
│     - Feedback, Änderungen, Verfeinerung                │
└─────────────────────────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────┐
│  3. VOTING (Abstimmung)                                  │
│     - Token-Holder stimmen ab                           │
│     - Meist über Snapshot oder On-Chain                 │
│     - Zeitfenster (z.B. 7 Tage)                         │
└─────────────────────────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────┐
│  4. EXECUTION (Ausführung)                               │
│     - Bei Erfolg: Smart Contract führt aus              │
│     - Treasury-Transfer, Parameter-Änderung, etc.       │
└─────────────────────────────────────────────────────────┘
```

### Die Probleme von DAOs (ChainSights-Relevanz)

| Problem | Beschreibung |
|---------|--------------|
| **Whale-Dominanz** | Wenige Großhalter kontrollieren Abstimmungen |
| **Geringe Beteiligung** | Oft stimmen <10% der Token-Holder ab |
| **Wallet-Fragmentierung** | Unklar, wie viele echte Menschen teilnehmen |
| **Plutokratie** | Geld = Macht, nicht 1 Person = 1 Stimme |
| **Sybil-Attacken** | Erstellen vieler Wallets für mehr Einfluss |

---

## 7. Governance in Web3

### Was bedeutet "Governance"?

> **Governance ist der Prozess, durch den eine Gemeinschaft kollektive Entscheidungen trifft — in Web3 meist durch Token-basierte Abstimmungen.**

### Governance-Mechanismen

**1. Token Voting (1 Token = 1 Vote)**
- Am häufigsten
- Problem: Whales dominieren
- Beispiel: Uniswap, Aave

**2. Quadratic Voting**
- Kosten steigen quadratisch: 1 Vote = 1 Token, 2 Votes = 4 Tokens
- Reduziert Whale-Macht
- Beispiel: Gitcoin

**3. Conviction Voting**
- Stimmen gewinnen Gewicht über Zeit
- Belohnt langfristiges Engagement
- Beispiel: 1Hive

**4. Delegation**
- Token-Holder delegieren Stimmrecht an Experten
- "Repräsentative Demokratie" für DAOs
- Beispiel: ENS, Compound

### Governance-Plattformen

| Plattform | Typ | Beschreibung |
|-----------|-----|--------------|
| **Snapshot** | Off-chain | Gaslose Abstimmungen, am weitesten verbreitet |
| **Tally** | On-chain | Echte On-Chain-Governance |
| **Boardroom** | Aggregator | Sammelt Governance über DAOs hinweg |
| **Commonwealth** | Forum + Voting | Diskussion und Abstimmung kombiniert |

### Das Governance-Problem visualisiert

```
Was die DAO SAGT:              Was die REALITÄT sein könnte:

"5.000 Wallets haben          "50 Personen mit je 100 Wallets
 abgestimmt!"                  haben abgestimmt"

 👤👤👤👤👤...                  👤👤👤👤👤
 (5.000 Personen?)              │ │ │ │ │
                                └─┴─┴─┴─┴── (je 100 Wallets)
```

**Das ist das Problem, das ChainSights sichtbar macht.**

---

## 8. Das Problem: Wallets lügen

### Die Kernthese von ChainSights

> **"Jede Metrik ist eine Lüge, bis du weißt, wer hinter der Wallet steckt."**

### Warum Wallet-Zahlen irreführend sind

**Szenario 1: Der Airdrop-Farmer**
```
Person erstellt 100 Wallets
→ Erhält 100x Airdrop
→ DAO zeigt "100 neue Mitglieder"
→ Realität: 1 Person
```

**Szenario 2: Der Whale, der sich versteckt**
```
Whale hat 10% aller Tokens
→ Will Konzentration verbergen
→ Verteilt auf 50 Wallets
→ DAO zeigt "kein Wallet über 0.2%"
→ Realität: 1 Entity kontrolliert 10%
```

**Szenario 3: Die Governance-Manipulation**
```
Angreifer will Proposal durchdrücken
→ Erstellt 1.000 Wallets
→ Stimmt 1.000x ab
→ DAO zeigt "überwältigende Mehrheit"
→ Realität: 1 Person hat manipuliert
```

### Was Konkurrenten zeigen vs. Was wichtig ist

| Was sie zeigen | Was relevant wäre |
|----------------|-------------------|
| "10.000 aktive Wallets" | "~2.500 unique Nutzer" |
| "500 Governance-Votes" | "47 echte Teilnehmer" |
| "Dezentrale Verteilung" | "3 Entities kontrollieren 60%" |
| "Wachstum: +200 Wallets" | "Wachstum: +12 neue Nutzer" |

### Die ChainSights-Lösung: Identity-Aware Analytics

**Schicht 1: Wallet-Clustering**
- Funding-Analyse (woher kam das Geld?)
- Timing-Korrelation (gleichzeitige Aktivität?)
- Interaktionsmuster (dieselben Contracts?)

**Schicht 2: Confidence Levels**
- "Hohe Konfidenz: Diese 5 Wallets = 1 Person"
- "Mittlere Konfidenz: Wahrscheinlich zusammengehörig"
- "Niedrige Konfidenz: Nicht genug Daten"

**Schicht 3: Ehrliche Metriken**
- "~47 unique Teilnehmer (±5)"
- "Dezentralisierungsgrad: 34%"
- "Whale-Konzentration: Hoch (3 Entities = 61%)"

---

## 9. On-Chain Analytics

### Was sind On-Chain Analytics?

> **On-Chain Analytics ist die Auswertung von Blockchain-Daten, um Muster, Verhalten und Trends zu verstehen.**

### Arten von On-Chain-Daten

| Datentyp | Beschreibung | Beispiel |
|----------|--------------|----------|
| **Transaktionen** | Transfers zwischen Wallets | 0x1... sendete 10 ETH an 0x2... |
| **Smart Contract Calls** | Interaktionen mit Contracts | 0x1... rief swap() auf Uniswap auf |
| **Token-Balances** | Wer hält wie viel | Wallet 0x1... hält 1.000 UNI |
| **Events/Logs** | Emittierte Ereignisse | VoteCast(voter, proposalId, support) |
| **Governance-Daten** | Proposals, Votes | Proposal #42: 60% Ja, 40% Nein |

### Warum ist alles öffentlich?

**Blockchain = Transparenz**

Jede Transaktion ist für jeden sichtbar. Das ist Feature, nicht Bug.

```
Transaktion auf Etherscan:

Von:     0x742d35Cc6634C0532925a3b844Bc9e7595f8B2E1
An:      0x1f9840a85d5aF5bf1D1762F925BDADdC4201F984 (Uniswap)
Wert:    0 ETH
Daten:   delegate(0x8f3a...)
Zeit:    2025-03-15 14:32:05 UTC
Gas:     0.002 ETH
```

**Jeder kann sehen:**
- Wer was wann getan hat
- Wie viel jemand besitzt
- Abstimmungsverhalten
- Transaktionshistorie

### Tools für On-Chain-Analyse

| Tool | Beschreibung | Stärke |
|------|--------------|--------|
| **Etherscan** | Block Explorer | Einzelne Transaktionen ansehen |
| **Dune Analytics** | SQL-basierte Dashboards | Flexible Abfragen |
| **Nansen** | Wallet Labels, Smart Money | Wer ist wer |
| **Glassnode** | Bitcoin/ETH Metriken | Marktindikatoren |
| **DeepDAO** | DAO-spezifisch | Governance-Überblick |

### Was ChainSights anders macht

| Andere Tools | ChainSights |
|--------------|-------------|
| Zeigen Wallet-Aktivität | Zeigen Menschen-Aktivität |
| Zählen Transaktionen | Zählen unique Teilnehmer |
| Labels für bekannte Wallets | Clustering für unbekannte Wallets |
| "Hier sind die Daten" | "Hier ist die Wahrheit dahinter" |

---

## 10. Die wichtigsten Akteure im Markt

### Analytics-Plattformen

**Dune Analytics**
- Gegründet: 2018 (Oslo, Norwegen)
- Modell: Community-erstellte SQL-Dashboards
- Stärke: 700.000+ Dashboards, 100+ Chains
- Schwäche: Erfordert SQL-Kenntnisse
- Nutzer: 6.000+ Web3-Teams

**Nansen**
- Gegründet: 2019 (Singapur)
- Modell: Wallet-Labels, Smart Money Tracking
- Stärke: 350M+ gelabelte Wallets
- Schwäche: Teuer, nicht EU-basiert
- Nutzer: Institutionelle Investoren, Funds

**Flipside Crypto**
- Gegründet: 2017 (Boston, USA)
- Modell: Community-Analytics mit Bounties
- Stärke: Komplett kostenlos
- Schwäche: Keine Wallet-Labels
- Nutzer: 170.000+ Analysten

**Glassnode**
- Gegründet: 2018 (Schweiz)
- Modell: Institutional-grade Metriken
- Stärke: Tiefe Bitcoin/ETH-Analyse
- Schwäche: Fokus auf Bitcoin, teuer
- Nutzer: Trader, Institutionen

**Chainalysis**
- Gegründet: 2014 (New York)
- Modell: Compliance & Investigations
- Stärke: Regierungskontakte, 75+ Länder
- Schwäche: Nicht für Endnutzer
- Nutzer: Behörden, Exchanges

### Governance-Tools

| Tool | Funktion | Nutzer |
|------|----------|--------|
| **Snapshot** | Off-chain Voting | Die meisten DAOs |
| **Tally** | On-chain Governance | Protokoll-DAOs |
| **Boardroom** | Governance Aggregator | Institutionen |
| **Karma** | Delegate Reputation | ENS, Optimism |

### Datenquellen

| Quelle | Beschreibung | Nutzung |
|--------|--------------|---------|
| **Covalent** | Unified API für 100+ Chains | ChainSights Datenquelle |
| **The Graph** | Dezentrales Indexing | Subgraphs für DApps |
| **Alchemy** | Node Infrastructure | DApp Development |
| **Infura** | Ethereum Nodes | API-Zugang |

---

## 11. DeFi — Dezentrale Finanzen

### Was ist DeFi?

> **DeFi (Decentralized Finance) umfasst Finanzdienstleistungen, die auf Blockchains laufen — ohne Banken, Broker oder traditionelle Vermittler.**

### DeFi vs. TradFi

| Aspekt | TradFi (Traditionell) | DeFi |
|--------|----------------------|------|
| **Börse** | NYSE, Xetra | Uniswap, Curve |
| **Kredit** | Bank | Aave, Compound |
| **Sparen** | Sparkonto | Yield Farming |
| **Derivate** | CME, Eurex | dYdX, GMX |
| **Zugang** | Bankkonto nötig | Nur Wallet nötig |
| **Öffnungszeiten** | Mo-Fr, 9-17 | 24/7/365 |

### Wichtige DeFi-Protokolle

| Protokoll | Typ | TVL (ca.) |
|-----------|-----|-----------|
| **Lido** | Liquid Staking | $30B+ |
| **Aave** | Lending | $15B+ |
| **Uniswap** | DEX | $5B+ |
| **MakerDAO** | Stablecoin | $8B+ |
| **Curve** | Stablecoin DEX | $3B+ |

### Warum ist DeFi relevant für ChainSights?

- **DeFi-Protokolle sind DAOs** — Governance ist kritisch
- **Whale-Problem** ist am größten in DeFi
- **TVL (Total Value Locked)** = Treasury = Budget für Analytics
- **Regulierung (MiCA)** trifft DeFi zuerst

---

## 12. NFTs — Non-Fungible Tokens

### Was sind NFTs?

> **NFTs sind einzigartige, nicht austauschbare Tokens auf der Blockchain — sie können Kunst, Sammlerstücke, Mitgliedschaften oder alles Einzigartige repräsentieren.**

### NFT Use Cases

| Kategorie | Beispiele |
|-----------|-----------|
| **Digitale Kunst** | Beeple, Art Blocks |
| **Sammlerstücke** | Bored Apes, CryptoPunks |
| **Gaming** | Axie Infinity, Gods Unchained |
| **Musik** | Royal, Sound.xyz |
| **Mitgliedschaften** | Token-gated Communities |
| **Identität** | ENS Namen, POAPs |

### NFTs und Governance

**Einige DAOs nutzen NFTs für Governance:**
- 1 NFT = 1 Vote (statt Token-basiert)
- Beispiel: Nouns DAO — jedes Noun NFT = 1 Stimme
- Weniger Whale-Problem (schwerer zu akkumulieren)

### Whale-Tracking bei NFTs

**ChainSights-Relevanz:**
- Wenige Wallets halten oft 80%+ einer Collection
- "Whale Alert" — wenn große Holder verkaufen
- Floor Price Manipulation durch konzentrierten Besitz

---

## 13. Regulierung: MiCA und die EU

### Was ist MiCA?

> **MiCA (Markets in Crypto-Assets) ist die EU-Verordnung zur Regulierung von Krypto-Assets — in Kraft seit 2024, vollständig anwendbar ab 2025.**

### MiCA-Kernpunkte

| Bereich | Anforderung |
|---------|-------------|
| **Stablecoins** | Reserve-Anforderungen, Whitepaper-Pflicht |
| **Service Provider** | Lizenzpflicht für Exchanges, Custodians |
| **Transparenz** | Offenlegung von Risiken, Governance |
| **Verbraucherschutz** | Beschwerdemechanismen, Haftung |

### Warum ist MiCA relevant für ChainSights?

**DAOs müssen Dezentralisierung nachweisen:**

```
Szenario: Ein DeFi-Protokoll behauptet, eine DAO zu sein

Regulierer fragt: "Wie dezentral seid ihr wirklich?"

Ohne ChainSights:     Mit ChainSights:
"Äh... wir haben      "Hier ist unser Governance Report:
 viele Token-Holder"   - 1.247 unique Teilnehmer
                       - Dezentralisierungsgrad: 67%
                       - Kein Entity über 8%
                       - MiCA-Bereitschaft: Hoch"
```

**ChainSights als Compliance-Tool:**
- Unabhängige Governance-Analyse
- Nachweisbare Dezentralisierung
- Dokumentation für Regulierer

### EU-Vorteil für ChainSights

| Faktor | Vorteil |
|--------|---------|
| **Standort Österreich** | Echte EU-Jurisdiktion |
| **NeonDB Frankfurt** | Daten in der EU |
| **DSGVO-Compliance** | Datenschutz ernst genommen |
| **MiCA-Verständnis** | Aus Versicherungsbranche |

---

## 14. Technische Infrastruktur

### Wie kommen On-Chain-Daten zu uns?

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│ Blockchain  │ ──► │ Indexer     │ ──► │ ChainSights │
│ (Ethereum)  │     │ (Covalent)  │     │ (Analyse)   │
└─────────────┘     └─────────────┘     └─────────────┘
     │                    │                    │
     │ Rohe Blöcke        │ Strukturierte      │ Identity-Aware
     │ & Transaktionen    │ API-Daten          │ Reports
```

### Datenquellen für ChainSights

**Covalent**
- Unified API für 100+ Blockchains
- Historische Daten
- Token-Balances, Transaktionen, NFTs
- Pricing: Freemium + Pay-per-Call

**The Graph**
- Dezentrales Indexing Protocol
- Subgraphs für spezifische Protokolle
- GraphQL-Abfragen
- Community-erstellt

**Snapshot API**
- Governance-Daten
- Proposals, Votes, Delegationen
- Kostenlos

### Tech Stack für ChainSights (geplant)

| Komponente | Technologie | Zweck |
|------------|-------------|-------|
| **Frontend** | Next.js | Landing Page |
| **Backend** | Vercel Serverless | API, Logik |
| **Datenbank** | NeonDB (Frankfurt) | Speicherung, DSGVO |
| **Datenquellen** | Covalent, The Graph | On-Chain-Daten |
| **AI/Analyse** | Claude API | Report-Generierung |
| **Reports** | PDF-Generation | Output |
| **Zahlung** | Stripe + Crypto | Fiat + USDC |

---

## 15. Glossar

### A-D

| Begriff | Erklärung |
|---------|-----------|
| **Airdrop** | Kostenlose Token-Verteilung, oft für Marketing |
| **AMM** | Automated Market Maker — DEX ohne Orderbuch |
| **APY** | Annual Percentage Yield — Jahreszins |
| **Block** | Einheit von Transaktionen auf der Blockchain |
| **Bridge** | Verbindung zwischen verschiedenen Blockchains |
| **Burn** | Token permanent zerstören (Reduzierung des Supply) |
| **CEX** | Centralized Exchange (Coinbase, Binance) |
| **Cold Wallet** | Offline-Wallet für sichere Aufbewahrung |
| **Consensus** | Mechanismus zur Einigung im Netzwerk |
| **DAO** | Decentralized Autonomous Organization |
| **DApp** | Decentralized Application |
| **DeFi** | Decentralized Finance |
| **DEX** | Decentralized Exchange (Uniswap) |
| **Delegate** | Stimmrecht an jemand anderen übertragen |

### E-L

| Begriff | Erklärung |
|---------|-----------|
| **ERC-20** | Standard für fungible Tokens auf Ethereum |
| **ERC-721** | Standard für NFTs auf Ethereum |
| **Etherscan** | Block Explorer für Ethereum |
| **Floor Price** | Niedrigster Preis einer NFT-Collection |
| **Fork** | Abspaltung einer Blockchain oder eines Projekts |
| **Gas** | Gebühr für Transaktionen auf Ethereum |
| **Governance** | Prozess der kollektiven Entscheidungsfindung |
| **Hash** | Eindeutiger digitaler Fingerabdruck |
| **HODL** | "Hold On for Dear Life" — langfristiges Halten |
| **Hot Wallet** | Online-Wallet, bequem aber weniger sicher |
| **Layer 1** | Basis-Blockchain (Ethereum, Bitcoin) |
| **Layer 2** | Skalierungslösung auf Layer 1 (Polygon, Arbitrum) |
| **Liquidity** | Verfügbare Handelstiefe |
| **LP** | Liquidity Provider — stellt Liquidität bereit |

### M-R

| Begriff | Erklärung |
|---------|-----------|
| **Mainnet** | Produktives Blockchain-Netzwerk |
| **Meme Coin** | Token ohne echten Nutzen, Community-getrieben |
| **Merkle Tree** | Datenstruktur für effiziente Verifikation |
| **Mint** | Neue Tokens oder NFTs erstellen |
| **Multi-Sig** | Wallet, die mehrere Signaturen benötigt |
| **NFT** | Non-Fungible Token — einzigartiger Token |
| **Node** | Computer, der das Netzwerk betreibt |
| **On-Chain** | Auf der Blockchain gespeichert/ausgeführt |
| **Off-Chain** | Außerhalb der Blockchain |
| **Oracle** | Dienst, der externe Daten auf die Blockchain bringt |
| **Private Key** | Geheimer Schlüssel zum Signieren von Transaktionen |
| **Proposal** | Vorschlag zur Abstimmung in einer DAO |
| **Protocol** | Regeln und Smart Contracts eines Projekts |
| **Public Key** | Öffentliche Adresse (abgeleitet vom Private Key) |
| **Rug Pull** | Betrug, bei dem Entwickler mit Geldern verschwinden |

### S-Z

| Begriff | Erklärung |
|---------|-----------|
| **Seed Phrase** | 12-24 Wörter zur Wallet-Wiederherstellung |
| **Slippage** | Preisänderung während einer Transaktion |
| **Smart Contract** | Selbstausführendes Programm auf der Blockchain |
| **Snapshot** | Off-chain Voting-Plattform für DAOs |
| **Staking** | Token sperren für Rewards oder Sicherheit |
| **Stablecoin** | Token mit stabilem Wert (meist $1) |
| **Sybil Attack** | Erstellen vieler Identitäten für Manipulation |
| **Testnet** | Test-Netzwerk für Entwicklung |
| **Token** | Digitale Einheit auf einer Blockchain |
| **TVL** | Total Value Locked — Gesamtwert in einem Protokoll |
| **Validator** | Node, der Transaktionen validiert (PoS) |
| **Wallet** | Schlüsselpaar zum Verwalten von Crypto-Assets |
| **Whale** | Großer Token-Holder mit signifikantem Einfluss |
| **Whitepaper** | Technisches Dokument eines Projekts |
| **Yield** | Rendite aus DeFi-Aktivitäten |
| **Zero Knowledge** | Kryptographische Methode zum Nachweis ohne Offenlegung |

---

## Zusammenfassung: Was du jetzt weißt

### Die Grundlagen

1. **Web3** = dezentrales Internet mit Eigentumsrechten
2. **Blockchain** = unveränderliches, transparentes Hauptbuch
3. **Smart Contracts** = automatisch ausgeführter Code
4. **Wallets** = deine Identität in Web3
5. **Tokens** = programmierbare digitale Einheiten

### Das Ökosystem

6. **DAOs** = dezentrale Organisationen mit Token-Governance
7. **Governance** = kollektive Entscheidungsfindung
8. **DeFi** = Finanzdienstleistungen ohne Banken
9. **NFTs** = einzigartige digitale Assets

### Das Problem

10. **Wallets lügen** = Metriken basieren auf Wallets, nicht Menschen
11. **Whale-Dominanz** = wenige kontrollieren viel
12. **Sybil-Risiko** = gefälschte Dezentralisierung

### Die Chance

13. **On-Chain Analytics** = alles ist öffentlich und analysierbar
14. **MiCA** = Regulierung schafft Nachfrage nach Compliance
15. **ChainSights** = Identity-Aware Analytics für die Wahrheit

---

## Nächste Schritte

| Aktion | Ressource |
|--------|-----------|
| Ethereum Basics vertiefen | ethereum.org/learn |
| Mit einer Wallet experimentieren | MetaMask installieren |
| Governance beobachten | snapshot.org durchstöbern |
| Twitter/X folgen | Accounts aus Presence Strategy |
| Eine DAO joinen | z.B. Bankless DAO (niedrige Einstiegshürde) |

---

*Dokument erstellt: Dezember 2025*  
*Zweck: Grundlagen für den Einstieg in Web3 Analytics*  
*Nächste Überarbeitung: Bei Bedarf nach Presence-Aufbau*
