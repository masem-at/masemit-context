# Story 1.4: Multi-Chain Wallet-Linking (SOL + ATOM)

Status: done

<!-- Note: Validation is optional. Run validate-create-story for quality check before dev-story. -->
<!-- Generated: 2026-02-21 | Epic: 1 | Story: 4 | Key: 1-4-multi-chain-wallet-linking-sol-atom -->

## Story

As a authentifizierter User (via ETH-Wallet oder Email),
I want meine Solana- und Cosmos-Wallets zusätzlich verknüpfen können,
So that ich mein komplettes Multi-Chain-Staking-Portfolio an einem Ort sehen kann.

## Acceptance Criteria

1. **Given** ein authentifizierter User (via ETH-Wallet oder Email) **When** er auf "Wallet hinzufügen" klickt **Then** sieht er einen Chain-Selector (SOL / ATOM)

2. **Given** ein User wählt "SOL" **When** der Solana Wallet Adapter sich öffnet (Phantom, Solflare, etc.) **Then** wird eine Message-Signatur angefordert **And** bei erfolgreicher Signatur wird die SOL-Wallet in der `linked_wallets`-Tabelle gespeichert **And** das Dashboard aktualisiert sich und zeigt die neue Chain

3. **Given** ein User wählt "ATOM" **When** Keplr Connect sich öffnet **Then** wird eine ADR-036 Message-Signatur angefordert **And** bei erfolgreicher Signatur wird die ATOM-Wallet in der `linked_wallets`-Tabelle gespeichert

4. **Given** ein User versucht eine bereits verknüpfte Wallet erneut zu verbinden **When** die Duplikat-Prüfung greift **Then** zeigt die UI einen Toast: "Diese Wallet ist bereits verbunden"

5. **Given** das Wallet-Linking fehlschlägt (Signatur abgelehnt, Timeout) **When** der Fehler erkannt wird **Then** zeigt die UI "Verbindung fehlgeschlagen" mit Retry-Option

6. **Given** ein User hat 2+ Chains verknüpft **When** das Dashboard geladen wird **Then** erscheint der Chain-Filter als Tabs ("Alle / ETH / SOL / ATOM") statt eines einfachen Banners

## Tasks / Subtasks

- [x] Task 1: DB-Schema — `linked_wallets`-Tabelle (AC: #2, #3, #4)
  - [x] 1.1: `linked_wallets`-Tabelle in `src/infrastructure/db/schema.ts` erstellen: id (uuid PK), user_id (FK → users.id, ON DELETE CASCADE), chain (varchar, check: 'eth'/'sol'/'atom'), wallet_address (varchar(100)), linked_at (timestamp with timezone, default NOW())
  - [x] 1.2: Unique Constraint auf `(chain, wallet_address)` — eine Wallet kann nur einmal über alle User verknüpft sein
  - [x] 1.3: Index `idx_linked_wallets_user_id` auf user_id
  - [x] 1.4: `drizzle-kit generate` für Migration-SQL
  - [x] 1.5: Migration auf NeonDB anwenden (`drizzle-kit migrate`)

- [x] Task 2: Linked-Wallet Repository (AC: #2, #3, #4)
  - [x] 2.1: `src/infrastructure/db/repositories/linked-wallet-repository.ts` erstellen
  - [x] 2.2: `linkWallet(userId: string, chain: string, walletAddress: string): Promise<LinkedWallet>` — INSERT, bei Duplikat spezifischen Error werfen
  - [x] 2.3: `findLinkedWalletsByUserId(userId: string): Promise<LinkedWallet[]>` — SELECT alle verknüpften Wallets
  - [x] 2.4: `findLinkedWalletByChainAndAddress(chain: string, walletAddress: string): Promise<LinkedWallet | null>` — Duplikat-Check
  - [x] 2.5: `unlinkWallet(id: string, userId: string): Promise<boolean>` — DELETE (für Story 1.5 vorbereitet)
  - [x] 2.6: `deleteAllLinkedWallets(userId: string): Promise<void>` — Cascade-Delete (für Story 1.6 vorbereitet)

- [x] Task 3: Solana Wallet Packages installieren (AC: #2)
  - [x] 3.1: `npm install --legacy-peer-deps @solana/wallet-adapter-react @solana/wallet-adapter-wallets @solana/wallet-adapter-base @solana/wallet-adapter-react-ui @solana/web3.js@^1.98.4 tweetnacl bs58`
  - [x] 3.2: Env-Vars ergänzen in `src/lib/env.ts`: `NEXT_PUBLIC_SOLANA_RPC_URL` (client, optional, default: mainnet-beta)
  - [x] 3.3: `.env.example` um Solana-Keys erweitern

- [x] Task 4: Cosmos/Keplr Packages installieren (AC: #3)
  - [x] 4.1: `npm install --legacy-peer-deps @keplr-wallet/types@^0.13.10 @keplr-wallet/cosmos@^0.12.282`
  - [x] 4.2: Env-Vars: Keine zusätzlichen nötig (Keplr nutzt eingebaute RPC-Endpoints)
  - [x] 4.3: Keplr Window-Typing: `src/types/keplr.d.ts` erstellen mit `declare global { interface Window { keplr?: Keplr } }`

- [x] Task 5: Solana Provider erstellen (AC: #2)
  - [x] 5.1: `src/providers/solana-provider.tsx` erstellen: `'use client'`, wraps `ConnectionProvider` + `WalletProvider` aus `@solana/wallet-adapter-react`
  - [x] 5.2: Wallets: `PhantomWalletAdapter`, `SolflareWalletAdapter` (+ Wallet Standard auto-detect)
  - [x] 5.3: `autoConnect: false` — User muss explizit verbinden
  - [x] 5.4: In `src/app/layout.tsx` einbinden: `<SolanaProvider>` innerhalb der Provider-Hierarchy, NACH `<AuthProvider>` und VOR `{children}`

- [x] Task 6: Signatur-Verification Server-Logik (AC: #2, #3)
  - [x] 6.1: `src/lib/auth/wallet-verification.ts` erstellen
  - [x] 6.2: `generateLinkingMessage(chain: string, walletAddress: string, nonce: string): string` — erzeugt die zu signierende Nachricht: `"StakeTrack AI möchte deine {chain}-Wallet verifizieren.\n\nWallet: {address}\nNonce: {nonce}\nDatum: {iso-date}"`
  - [x] 6.3: `verifySolanaSignature(message: string, signature: Uint8Array, publicKey: string): boolean` — nutzt `tweetnacl.sign.detached.verify()` mit `bs58.decode(publicKey)`
  - [x] 6.4: `verifyCosmosSignature(message: string, signature: StdSignature, signerAddress: string): boolean` — nutzt `verifyADR36Amino()` aus `@keplr-wallet/cosmos`
  - [x] 6.5: Nonce-Management: Nutze NextAuth CSRF-Token als Nonce (gleicher Pattern wie SIWE in Story 1-2)

- [x] Task 7: Wallet-Linking API Endpoint (AC: #2, #3, #4, #5)
  - [x] 7.1: `src/app/api/wallets/link/route.ts` — POST: Empfängt `{ chain, walletAddress, message, signature }`, verifiziert Session (authentifizierter User), verifiziert Signatur per Chain, prüft Duplikat, speichert in `linked_wallets`
  - [x] 7.2: Zod-Schema für Request-Validierung: chain (enum 'sol'|'atom'), walletAddress (string min 32 max 100), message (string), signature (string, base64-encoded)
  - [x] 7.3: Response Format: `{ data: { id, chain, walletAddress, linkedAt } }` (Success), `{ error, code }` (Error)
  - [x] 7.4: Error Codes: `DUPLICATE_WALLET` (409), `INVALID_SIGNATURE` (401), `UNAUTHORIZED` (401), `VALIDATION_ERROR` (400)
  - [x] 7.5: Logging: `[wallet-link] SOL wallet linked for user [userId-kurz]` — KEINE Wallet-Adressen in Logs (NFR15)

- [x] Task 8: Wallet-Listing API Endpoint (AC: #6)
  - [x] 8.1: `src/app/api/wallets/route.ts` — GET: Gibt alle verknüpften Wallets des authentifizierten Users zurück
  - [x] 8.2: Response: `{ data: LinkedWallet[], meta: { count: number } }`
  - [x] 8.3: Inkludiert auch die primäre ETH-Wallet aus `users.wallet_address` (falls vorhanden) als virtuelle LinkedWallet mit `chain: 'eth'`

- [x] Task 9: Settings-Page Grundgerüst (AC: #1, #2, #3, #4, #5)
  - [x] 9.1: `src/app/(dashboard)/settings/page.tsx` erstellen — RSC-Page mit Titel "Einstellungen"
  - [x] 9.2: Abschnitt "Verknüpfte Wallets" — zeigt Liste aller Wallets (primäre + linked)
  - [x] 9.3: Pro Wallet: Chain-Badge (ETH/SOL/ATOM), gekürzte Adresse, Verknüpfungsdatum
  - [x] 9.4: Primäre Auth-Wallet als "Primär" markiert (nicht entfernbar — Story 1.5)
  - [x] 9.5: "Wallet hinzufügen"-Button unterhalb der Liste
  - [x] 9.6: Middleware erweitern: `/settings` und `/settings/:path*` in matcher aufnehmen

- [x] Task 10: Chain-Selector & SOL-Linking UI (AC: #1, #2, #5)
  - [x] 10.1: `src/components/settings/AddWalletDialog.tsx` erstellen — Modal/Bottom Sheet mit Chain-Auswahl
  - [x] 10.2: Chain-Selector: Zwei Buttons "Solana" und "Cosmos", jeweils mit Chain-Logo
  - [x] 10.3: SOL-Linking-Flow: Bei Klick auf "Solana" → `useWallet()` Hook aus `@solana/wallet-adapter-react`, Wallet-Modal öffnen, nach Connect → `wallet.signMessage()` mit generiertem Linking-Message aufrufen, Signatur an `POST /api/wallets/link` senden
  - [x] 10.4: Loading-State während Signatur-Verification
  - [x] 10.5: Success-Toast: "Solana-Wallet verknüpft!" — TanStack Query Cache invalidieren
  - [x] 10.6: Error-States: Signatur abgelehnt ("Signatur nötig um Wallet zu verifizieren"), Timeout ("Verbindung fehlgeschlagen. Nochmal versuchen?"), Duplikat ("Diese Wallet ist bereits verbunden")

- [x] Task 11: ATOM-Linking UI (AC: #1, #3, #5)
  - [x] 11.1: ATOM-Linking-Flow in `AddWalletDialog.tsx`: Bei Klick auf "Cosmos" → prüfe `window.keplr` Verfügbarkeit, `keplr.enable('cosmoshub-4')`, `keplr.getKey('cosmoshub-4')` für Adresse, `keplr.signArbitrary('cosmoshub-4', address, linkingMessage)` für ADR-036 Signatur
  - [x] 11.2: Keplr-nicht-installiert-Hinweis: "Keplr Wallet Extension wird benötigt. → Installieren" mit Link
  - [x] 11.3: Signatur an `POST /api/wallets/link` senden (chain: 'atom')
  - [x] 11.4: Success-Toast: "Cosmos-Wallet verknüpft!"
  - [x] 11.5: Error-States analog zu SOL

- [x] Task 12: Dashboard Chain-Filter erweitern (AC: #6)
  - [x] 12.1: Chain-Filter-Tabs im Dashboard erweitern: Aktuell nur UI-Shell vorhanden, muss linked Wallets fetchen um relevante Chain-Tabs dynamisch anzuzeigen
  - [x] 12.2: `useLinkedWallets()` Hook erstellen in `src/lib/hooks/use-linked-wallets.ts` — TanStack Query, fetcht `GET /api/wallets`
  - [x] 12.3: Chain-Filter zeigt nur Tabs für Chains die der User tatsächlich verknüpft hat + "Alle"
  - [x] 12.4: Bei nur 1 Chain: Kein Filter anzeigen (nur Content)

- [x] Task 13: Tests (AC: alle)
  - [x] 13.1: `src/infrastructure/db/repositories/linked-wallet-repository.test.ts` — CRUD-Operationen, Duplikat-Check, Cascade-Delete — 8 Tests
  - [x] 13.2: `src/lib/auth/wallet-verification.test.ts` — SOL-Signatur-Verification (valid, invalid, wrong key), ATOM-Signatur-Verification (valid, invalid), Message-Generation — 8 Tests
  - [x] 13.3: `src/app/api/wallets/link/route.test.ts` — API-Endpoint: Auth-Check, Validation, Duplicate, Success SOL, Success ATOM — 7 Tests
  - [x] 13.4: `src/app/api/wallets/route.test.ts` — GET: Auth-Check, leere Liste, mit Wallets, primäre ETH inkludiert — 4 Tests
  - [x] 13.5: `src/infrastructure/db/schema.test.ts` erweitern — linked_wallets-Tabelle Schema-Validierung — 2 Tests
  - [ ] 13.6: Dev-Server Smoke-Test: Kompletter SOL+ATOM Linking-Flow manuell verifizieren

## Dev Notes

### Kritische Architektur-Entscheidungen

- **Wallet-Linking ist KEIN neuer NextAuth-Provider:** SOL/ATOM-Linking geschieht POST-Login durch einen bereits authentifizierten User. Es wird ein separater API-Endpoint `POST /api/wallets/link` verwendet, NICHT ein neuer CredentialsProvider. Die Authentifizierung bleibt bei ETH-SIWE oder Email. Multi-Chain-Wallets sind sekundäre Datenquellen, keine Auth-Methoden.

- **Solana Legacy Adapter Stack (NICHT @solana/kit):** `@solana/wallet-adapter-react` 0.15.39 ist die stabile, weit verbreitete Lösung. `@solana/kit` 6.1.0 ist zwar neuer, aber ein komplett anderes API-Design (functional statt class-based) und hat weniger Ökosystem-Support. Für Wallet-Linking (Connect + signMessage) reicht der Legacy-Stack vollkommen.

- **Keplr direkt via window.keplr (KEIN cosmos-kit):** `@cosmos-kit/react` wäre Overkill für reines Wallet-Linking (Connect + signArbitrary). Stattdessen: Direkte Keplr Browser Extension API via `window.keplr`. Nur `@keplr-wallet/types` für TypeScript-Typen und `@keplr-wallet/cosmos` für Server-seitige ADR-036 Signatur-Verifizierung. Spart ~200KB Bundle-Size.

- **Solana @solana/web3.js 1.98.4 (NICHT 1.95.6/1.95.7):** Versionen 1.95.6 und 1.95.7 waren von einer Supply-Chain-Attack betroffen (Dezember 2024). Version 1.98.4 ist die aktuelle sichere 1.x-Version. NIEMALS 1.95.6 oder 1.95.7 verwenden!

- **linked_wallets.wallet_address varchar(100):** ETH-Adressen sind 42 Zeichen (0x + 40 hex), SOL-Adressen bis 44 Zeichen (base58), ATOM-Adressen bis ~46 Zeichen (bech32 `cosmos1...`). varchar(100) deckt alle Chains mit Reserve ab.

- **Nonce via NextAuth CSRF-Token:** Gleicher Pattern wie SIWE in Story 1-2. Die CSRF-Token-Nonce verhindert Replay-Attacks ohne eigenes Nonce-Management.

- **Primäre ETH-Wallet aus users-Tabelle:** Die primäre Auth-Wallet (ETH) steht in `users.wallet_address`. Die `linked_wallets`-Tabelle enthält NUR sekundäre Wallets (SOL, ATOM, und zukünftig weitere ETH-Wallets). Der GET `/api/wallets` Endpoint kombiniert beide Quellen zu einer einheitlichen Liste.

- **SolanaProvider Position in Provider-Hierarchy:** `<SolanaProvider>` muss NACH `<AuthProvider>` und `<Web3Provider>` stehen, da es unabhängig von RainbowKit/wagmi ist. Es ist ein separater Provider-Stack nur für SOL-Wallet-Interaktionen (Linking, nicht Auth).

### DB-Schema: `linked_wallets`-Tabelle

```sql
CREATE TABLE linked_wallets (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  chain VARCHAR(10) NOT NULL CHECK (chain IN ('eth', 'sol', 'atom')),
  wallet_address VARCHAR(100) NOT NULL,
  linked_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
  CONSTRAINT uq_linked_wallets_chain_address UNIQUE (chain, wallet_address)
);

CREATE INDEX idx_linked_wallets_user_id ON linked_wallets (user_id);
```

**Drizzle-Schema (TypeScript):**
```typescript
export const linkedWallets = pgTable(
  "linked_wallets",
  {
    id: uuid("id").primaryKey().defaultRandom(),
    userId: uuid("user_id").notNull().references(() => users.id, { onDelete: "cascade" }),
    chain: varchar("chain", { length: 10 }).notNull(),
    walletAddress: varchar("wallet_address", { length: 100 }).notNull(),
    linkedAt: timestamp("linked_at", { withTimezone: true }).notNull().defaultNow(),
  },
  (table) => [
    uniqueIndex("uq_linked_wallets_chain_address").on(table.chain, table.walletAddress),
    index("idx_linked_wallets_user_id").on(table.userId),
  ],
);
```

### Wallet-Linking API Pattern

```typescript
// POST /api/wallets/link
// Request Body:
{
  chain: "sol" | "atom",
  walletAddress: string,
  message: string,
  signature: string  // base64-encoded
}

// Success Response:
{
  data: {
    id: string,
    chain: string,
    walletAddress: string,
    linkedAt: string
  }
}

// Error Responses:
{ error: "Diese Wallet ist bereits verbunden", code: "DUPLICATE_WALLET" }      // 409
{ error: "Ungültige Signatur", code: "INVALID_SIGNATURE" }                     // 401
{ error: "Nicht authentifiziert", code: "UNAUTHORIZED" }                       // 401
{ error: "Validierungsfehler", code: "VALIDATION_ERROR", details: ZodError }   // 400
```

### Solana Signatur-Verification (Server-Side)

```typescript
import nacl from "tweetnacl";
import bs58 from "bs58";

export function verifySolanaSignature(
  message: string,
  signatureBase64: string,
  publicKeyBase58: string,
): boolean {
  const messageBytes = new TextEncoder().encode(message);
  const signatureBytes = Buffer.from(signatureBase64, "base64");
  const publicKeyBytes = bs58.decode(publicKeyBase58);

  return nacl.sign.detached.verify(messageBytes, signatureBytes, publicKeyBytes);
}
```

### Cosmos ADR-036 Signatur-Verification (Server-Side)

```typescript
import { verifyADR36Amino } from "@keplr-wallet/cosmos";

export function verifyCosmosSignature(
  message: string,
  signature: { pub_key: { type: string; value: string }; signature: string },
  signerAddress: string,
): boolean {
  const messageBytes = new TextEncoder().encode(message);
  const pubKeyBytes = Buffer.from(signature.pub_key.value, "base64");
  const sigBytes = Buffer.from(signature.signature, "base64");

  return verifyADR36Amino(
    "cosmos",         // bech32 prefix
    signerAddress,
    messageBytes,
    pubKeyBytes,
    sigBytes,
  );
}
```

### Linking-Message Format

```
StakeTrack AI möchte deine Solana-Wallet verifizieren.

Wallet: 7xKX...4bPq
Nonce: abc123def456
Datum: 2026-02-21T14:30:00Z

Dieser Vorgang gewährt Read-Only-Zugang zu deinen Staking-Daten.
```

### SOL-Linking Client-Flow

```typescript
// In AddWalletDialog.tsx — Solana Flow
import { useWallet } from "@solana/wallet-adapter-react";
import { useWalletModal } from "@solana/wallet-adapter-react-ui";

const { publicKey, signMessage, connected, disconnect } = useWallet();
const { setVisible } = useWalletModal();

// 1. User klickt "Solana" → Wallet-Modal öffnen
setVisible(true);

// 2. Nach Connect: Message generieren (mit Nonce vom Server)
const nonce = await getCsrfToken(); // NextAuth CSRF Token
const message = `StakeTrack AI möchte deine Solana-Wallet verifizieren.\n\nWallet: ${publicKey.toBase58()}\nNonce: ${nonce}\nDatum: ${new Date().toISOString()}\n\nDieser Vorgang gewährt Read-Only-Zugang zu deinen Staking-Daten.`;

// 3. Signieren
const messageBytes = new TextEncoder().encode(message);
const signature = await signMessage(messageBytes);

// 4. An API senden
const res = await fetch("/api/wallets/link", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({
    chain: "sol",
    walletAddress: publicKey.toBase58(),
    message,
    signature: Buffer.from(signature).toString("base64"),
  }),
});

// 5. Disconnect Solana-Wallet nach Linking (nicht als Session-Auth verwenden)
disconnect();
```

### ATOM-Linking Client-Flow

```typescript
// In AddWalletDialog.tsx — Cosmos Flow (direkte Keplr API)

// 1. Keplr verfügbar?
if (!window.keplr) {
  // Zeige "Keplr installieren" Hinweis
  return;
}

// 2. Keplr enable + Key holen
await window.keplr.enable("cosmoshub-4");
const key = await window.keplr.getKey("cosmoshub-4");
const address = key.bech32Address;

// 3. Message generieren
const nonce = await getCsrfToken();
const message = `StakeTrack AI möchte deine Cosmos-Wallet verifizieren.\n\nWallet: ${address}\nNonce: ${nonce}\nDatum: ${new Date().toISOString()}\n\nDieser Vorgang gewährt Read-Only-Zugang zu deinen Staking-Daten.`;

// 4. ADR-036 Signatur
const signature = await window.keplr.signArbitrary("cosmoshub-4", address, message);

// 5. An API senden
const res = await fetch("/api/wallets/link", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({
    chain: "atom",
    walletAddress: address,
    message,
    signature: JSON.stringify(signature), // StdSignature als JSON-String
  }),
});
```

### SolanaProvider Pattern

```typescript
// src/providers/solana-provider.tsx
"use client";

import { ConnectionProvider, WalletProvider } from "@solana/wallet-adapter-react";
import { WalletModalProvider } from "@solana/wallet-adapter-react-ui";
import { PhantomWalletAdapter, SolflareWalletAdapter } from "@solana/wallet-adapter-wallets";
import { clusterApiUrl } from "@solana/web3.js";
import { useMemo } from "react";

import "@solana/wallet-adapter-react-ui/styles.css";

export function SolanaProvider({ children }: { children: React.ReactNode }) {
  const endpoint = process.env.NEXT_PUBLIC_SOLANA_RPC_URL ?? clusterApiUrl("mainnet-beta");

  const wallets = useMemo(
    () => [new PhantomWalletAdapter(), new SolflareWalletAdapter()],
    [],
  );

  return (
    <ConnectionProvider endpoint={endpoint}>
      <WalletProvider wallets={wallets} autoConnect={false}>
        <WalletModalProvider>
          {children}
        </WalletModalProvider>
      </WalletProvider>
    </ConnectionProvider>
  );
}
```

### Provider-Hierarchy nach Story 1.4

```
<html> (layout.tsx)
  <Web3Provider>             ← RainbowKit + wagmi (ETH) — MUSS outermost sein
    <AuthProvider>           ← NextAuth SessionProvider + RainbowKit SIWE
      <SolanaProvider>       ← NEU: Solana Wallet Adapter
        <ThemeProvider>      ← Dark/Light Toggle — innermost
          {children}
        </ThemeProvider>
      </SolanaProvider>
    </AuthProvider>
  </Web3Provider>
</html>
```

### Settings-Page UI-Struktur

```
┌───────────────────────────────────┐
│  Einstellungen                     │
│                                    │
│  ── Verknüpfte Wallets ──         │
│                                    │
│  ┌─────────────────────────────┐  │
│  │ ETH  0x1234...5678  Primär  │  │  ← Aus users.wallet_address
│  │      Verknüpft am 18.02.26 │  │
│  ├─────────────────────────────┤  │
│  │ SOL  7xKX...4bPq           │  │  ← Aus linked_wallets
│  │      Verknüpft am 21.02.26 │  │
│  ├─────────────────────────────┤  │
│  │ ATOM cosmos1...abc4         │  │  ← Aus linked_wallets
│  │      Verknüpft am 21.02.26 │  │
│  └─────────────────────────────┘  │
│                                    │
│  [ + Wallet hinzufügen ]          │  ← Öffnet AddWalletDialog
│                                    │
│  ── Account ──                    │  ← Vorbereitet für Story 1.5/1.6
│  (Wallet-Verwaltung, Löschung)    │
└───────────────────────────────────┘
```

### Paketversionen (Stand: 2026-02-21)

| Paket | Version | Hinweis |
|---|---|---|
| `@solana/wallet-adapter-react` | ~0.15.39 | Stabile Legacy-API |
| `@solana/wallet-adapter-wallets` | ~0.19.37 | Phantom + Solflare Adapter |
| `@solana/wallet-adapter-base` | ~0.9.24 | Peer-Dependency |
| `@solana/wallet-adapter-react-ui` | ~0.9.39 | Wallet-Modal-UI |
| `@solana/web3.js` | ^1.98.4 | NICHT 1.95.6/7 (Supply-Chain-Attack!) |
| `tweetnacl` | ~1.0.3 | ed25519 Signatur-Verify (SOL) |
| `bs58` | ~6.0.0 | Base58 Encoding/Decoding (SOL) |
| `@keplr-wallet/types` | ^0.13.10 | TypeScript-Typen für Keplr API |
| `@keplr-wallet/cosmos` | ^0.12.282 | ADR-036 Server-Verify |

### Learnings aus Story 1-2 und 1-3

- **`--legacy-peer-deps` nötig:** Für ALLE npm installs wegen React 19 Peer-Dep-Konflikten. IMMER verwenden.
- **Provider-Reihenfolge KRITISCH:** `SolanaProvider` NACH `AuthProvider` und `Web3Provider` einfügen. Keine verschachtelte Abhängigkeit, aber die Reihenfolge muss konsistent sein.
- **Drizzle `neon-http` Modus:** HTTP-basierter Treiber. Für Wallet-Linking reichen einzelne INSERT/SELECT Queries (keine Transaktion nötig).
- **Biome 2.2.0:** Alle neuen Dateien müssen Biome-Check bestehen.
- **Next.js 15.5.12 (LTS), Tailwind v4:** Custom Tokens in `globals.css` via `@theme`.
- **Wallet-Login bleibt ETH-Only:** SOL/ATOM sind KEINE Auth-Methoden, nur sekundäre Datenquellen.
- **`truncateAddress()` in `src/lib/utils/format.ts`:** Existiert bereits — für Wallet-Adressen-Kürzung in UI und Logs verwenden.
- **Resend statt MMS:** MMS hat keinen Email-Endpoint. Nicht relevant für diese Story, aber als Kontext.
- **In-Memory Rate-Limiting:** Upstash Rate Limit nicht installiert. Falls Rate-Limiting auf `/api/wallets/link` nötig, gleichen In-Memory-Fallback wie in Story 1-3 verwenden.
- **serverEnv statt process.env:** IMMER `serverEnv` aus `@/lib/env` verwenden, NICHT `process.env` direkt (Review-Finding H2 aus Story 1-3).

### Bestehende Infrastruktur (aus Story 1-1, 1-2, 1-3)

| Komponente | Datei | Status |
|---|---|---|
| DB Client | `src/infrastructure/db/index.ts` | Existiert (Drizzle + neon-http) |
| DB Schema | `src/infrastructure/db/schema.ts` | Existiert (users + verification_tokens) → MUSS um linked_wallets erweitert werden |
| User Repository | `src/infrastructure/db/repositories/user-repository.ts` | Existiert (findByWallet, findByEmail, upsert) → NICHT ändern |
| Env Validation | `src/lib/env.ts` | Existiert → MUSS um SOLANA_RPC_URL erweitert werden |
| NextAuth Config | `src/lib/auth/auth-options.ts` | Existiert (SIWE + Email) → NICHT ändern |
| Middleware | `src/middleware.ts` | Existiert → MUSS `/settings` in matcher aufnehmen |
| Web3 Provider | `src/providers/web3-provider.tsx` | Existiert (ETH-only RainbowKit) → NICHT ändern |
| Auth Provider | `src/providers/auth-provider.tsx` | Existiert → NICHT ändern |
| Root Layout | `src/app/layout.tsx` | Existiert → MUSS SolanaProvider einbinden |
| Sidebar/Header/BottomNav | `src/components/layout/` | Existieren mit `/settings`-Link → NICHT ändern |
| Format Utils | `src/lib/utils/format.ts` | `truncateAddress()` vorhanden |
| Dashboard Page | `src/app/(dashboard)/page.tsx` | Existiert → Optional: Chain-Filter erweitern (Task 12) |
| Settings Page | `src/app/(dashboard)/settings/page.tsx` | EXISTIERT NICHT → MUSS erstellt werden |

### Sicherheitsaspekte

- **Server-seitige Signatur-Verifizierung (PFLICHT):** Sowohl SOL (tweetnacl) als auch ATOM (verifyADR36Amino) Signaturen werden ausschließlich auf dem Server verifiziert. Client-Side-Only-Verification ist NICHT ausreichend.
- **Nonce-basierter Replay-Schutz:** Jede Linking-Nachricht enthält eine server-generierte Nonce (NextAuth CSRF-Token). Verhindert Replay-Attacks.
- **Domain-Binding:** Die Linking-Message enthält "StakeTrack AI" als Domain-Identifikator. Verhindert Cross-Site-Replay.
- **Unique Constraint (chain, wallet_address):** Eine Wallet kann nur einmal systemweit verknüpft werden. Verhindert, dass ein Angreifer eine fremde Wallet seinem Account zuordnet (Signatur-Beweis nötig).
- **Keine PII in Logs (NFR15):** Wallet-Adressen nur gekürzt loggen, User-IDs nur gekürzt.
- **ON DELETE CASCADE:** Wenn ein User gelöscht wird (Story 1.6 DSGVO), werden automatisch alle linked_wallets gelöscht.
- **@solana/web3.js 1.98.4:** KEINE Version 1.95.6/7 verwenden (Supply-Chain-Attack Dezember 2024).

### Abgrenzung: Was diese Story NICHT macht

- **Kein Wallet-Entfernen:** Das ist Story 1.5 (Wallet-Verwaltung & Einstellungen). Die Repository-Methoden `unlinkWallet()` und `deleteAllLinkedWallets()` werden vorbereitet aber die UI dafür kommt in 1.5.
- **Kein Account-Löschen:** Das ist Story 1.6 (Account-Löschung DSGVO). Der CASCADE DELETE auf der DB-Ebene ist durch `ON DELETE CASCADE` bereits abgesichert.
- **Keine Staking-Daten-Anzeige:** Die verknüpften SOL/ATOM-Wallets werden gespeichert, aber Staking-Positionen werden erst in Epic 2 (Chain Adapters) abgerufen.
- **Kein Tier-Gating:** Free-User dürfen in dieser Story noch alle Chains verknüpfen. Das 1-Chain-Limit für Free-User kommt in Epic 6 (Freemium-Gating).

### Project Structure Notes

- `linked-wallet-repository.ts` gehört in `src/infrastructure/db/repositories/` (Infrastructure Layer)
- `wallet-verification.ts` gehört in `src/lib/auth/` (Shared Utilities — Signatur-Verify ist kein Domain-Logic)
- `solana-provider.tsx` gehört in `src/providers/` (Provider Layer)
- API Routes gehören in `src/app/api/wallets/` (App Layer)
- Settings-Page gehört in `src/app/(dashboard)/settings/` (App Layer, Dashboard Route Group)
- `AddWalletDialog.tsx` gehört in `src/components/settings/` (Komponenten-Layer)
- `use-linked-wallets.ts` Hook gehört in `src/lib/hooks/` (Shared Utilities)
- Keplr Window-Typing gehört in `src/types/` (TypeScript-Typen)
- Domain Layer (`src/domain/`) wird in dieser Story NICHT berührt

### References

- [Source: _bmad-output/planning-artifacts/epics.md#Story 1.4 — Multi-Chain Wallet-Linking]
- [Source: _bmad-output/planning-artifacts/epics.md#Epic 1 — FR2, FR3, FR5]
- [Source: _bmad-output/planning-artifacts/architecture.md#Authentication & Security]
- [Source: _bmad-output/planning-artifacts/architecture.md#Core Architectural Decisions]
- [Source: _bmad-output/planning-artifacts/architecture.md#Implementation Patterns & Consistency Rules]
- [Source: _bmad-output/planning-artifacts/architecture.md#Complete Project Directory Structure]
- [Source: _bmad-output/planning-artifacts/architecture.md#Architectural Boundaries]
- [Source: _bmad-output/planning-artifacts/prd.md#FR2 — SOL-Wallet-Connect + Message-Signing]
- [Source: _bmad-output/planning-artifacts/prd.md#FR3 — ATOM-Wallet-Connect + ADR-036]
- [Source: _bmad-output/planning-artifacts/prd.md#FR5 — Multi-Wallet-Linking]
- [Source: _bmad-output/planning-artifacts/prd.md#NFR10 — Keine Wallet Private Keys]
- [Source: _bmad-output/planning-artifacts/prd.md#NFR15 — Keine PII in Logs]
- [Source: _bmad-output/project-context.md#Auth Stack — Keplr, Solana Wallet]
- [Source: _bmad-output/project-context.md#Authentication Rules — Unified User Profile, Multi-Wallet]
- [Source: _bmad-output/project-context.md#Security Rules — Read-Only Analytics, keine Private Keys]
- [Source: _bmad-output/project-context.md#Solana-spezifische Warnung — getStakeActivation deprecated]
- [Source: _bmad-output/implementation-artifacts/1-3-email-authentifizierung-via-mms.md — Learnings, File List]
- [Source: _bmad-output/implementation-artifacts/1-2-eth-wallet-authentifizierung-siwe.md — Referenced via 1-3 Learnings]
- [Web: Solana Wallet Adapter — github.com/anza-xyz/wallet-adapter — v0.15.39]
- [Web: @solana/web3.js Supply Chain Attack — Versionen 1.95.6/7 kompromittiert, 1.98.4 sicher]
- [Web: Keplr signArbitrary API — docs.keplr.app/api/guide/sign-arbitrary]
- [Web: ADR-036 Cosmos Arbitrary Signing — github.com/cosmos/cosmos-sdk/blob/main/docs/architecture/adr-036-arbitrary-signature.md]
- [Web: @keplr-wallet/cosmos verifyADR36Amino — npmjs.com/package/@keplr-wallet/cosmos v0.12.282]
- [Web: SIWS Sign In With Solana — siws.web3auth.io/spec]
- [Web: tweetnacl ed25519 — npmjs.com/package/tweetnacl v1.0.3]

## Dev Agent Record

### Agent Model Used

Claude Opus 4.6 (claude-opus-4-6)

### Debug Log References

### Completion Notes List

- ✅ Task 1: `linked_wallets` Drizzle-Schema mit Unique Constraint, Index und CHECK Constraint erstellt. Migration `0003_faithful_natasha_romanoff.sql` generiert und auf NeonDB angewendet. 2 neue Schema-Tests bestehen.
- ✅ Task 2: Repository mit 6 Funktionen implementiert (linkWallet, findLinkedWalletsByUserId, findLinkedWalletByChainAndAddress, unlinkWallet, deleteAllLinkedWallets) inkl. DuplicateWalletError-Klasse. 8 Unit-Tests bestehen.
- ✅ Task 3: Solana Wallet Packages installiert (@solana/wallet-adapter-react, @solana/wallet-adapter-wallets, @solana/wallet-adapter-base, @solana/wallet-adapter-react-ui, @solana/web3.js@1.98.4, tweetnacl, bs58). NEXT_PUBLIC_SOLANA_RPC_URL in env.ts und .env.example ergänzt.
- ✅ Task 4: Cosmos/Keplr Packages installiert (@keplr-wallet/types, @keplr-wallet/cosmos). Keplr Window-Typing in src/types/keplr.d.ts erstellt.
- ✅ Task 5: SolanaProvider mit PhantomWalletAdapter + SolflareWalletAdapter, autoConnect: false. In layout.tsx nach AuthProvider eingebunden.
- ✅ Task 6: wallet-verification.ts mit generateLinkingMessage, verifySolanaSignature (tweetnacl), verifyCosmosSignature (ADR-036). 8 Unit-Tests.
- ✅ Task 7: POST /api/wallets/link — Zod-Validierung, Chain-basierte Signatur-Verification, DuplicateWallet-Handling, NFR15-konformes Logging. 7 Tests.
- ✅ Task 8: GET /api/wallets — Gibt linked Wallets + primäre ETH-Wallet zurück. 4 Tests. Middleware um /settings erweitert.
- ✅ Task 9: Settings-Page als RSC mit Wallet-Liste (Chain-Badge, gekürzte Adresse, Datum, Primär-Markierung). AddWalletDialog-Button.
- ✅ Task 10: AddWalletDialog mit Chain-Selector (SOL/ATOM), SOL-Linking via @solana/wallet-adapter-react, State-Management (idle/connecting/signing/verifying/success/error), TanStack Query Cache Invalidierung.
- ✅ Task 11: ATOM-Linking im AddWalletDialog — direkte Keplr window.keplr API, ADR-036 signArbitrary, Keplr-nicht-installiert Hinweis.
- ✅ Task 12: ChainFilter-Komponente mit dynamischen Tabs basierend auf verknüpften Chains. useLinkedWallets Hook (TanStack Query). Bei ≤1 Chain: kein Filter.
- ✅ Task 13: Alle automatisierten Tests bestehen (89 Tests). 13.6 (manueller Smoke-Test) offen für User.

### Change Log

- 2026-02-21: Tasks 1-4 implementiert — DB-Schema, Repository, Solana + Keplr Packages (70 Tests bestehen, keine Regressionen)
- 2026-02-21: Tasks 5-8 implementiert — SolanaProvider, Signatur-Verification, Wallet-Linking API, Wallet-Listing API, Middleware /settings (89 Tests bestehen, keine Regressionen)
- 2026-02-21: Tasks 9-13 implementiert — Settings-Page, AddWalletDialog (SOL+ATOM), ChainFilter, useLinkedWallets Hook (89 Tests bestehen, keine Regressionen). Story auf "review" gesetzt.

## Senior Developer Review (AI)

**Reviewer:** Sempre (via Claude Opus 4.6 adversarial review)
**Date:** 2026-02-21
**Outcome:** Approved (after fixes)

### Issues Found: 3 High, 3 Medium, 3 Low

**Fixed (6 issues):**

| # | Severity | Issue | Fix |
|---|----------|-------|-----|
| H1 | HIGH | Nonce wird serverseitig NICHT validiert — Replay-Attack moeglich | Server extrahiert Nonce aus Message, validiert gegen CSRF-Token aus Cookies. 2 neue Tests (nonce-missing, nonce-mismatch) |
| H2 | HIGH | JSON.parse(signature) fuer ATOM ohne Validierung — Server-Crash-Vektor | try/catch + Zod-Schema `cosmosSignatureSchema` fuer CosmosStdSignature Validierung. Ungültiges JSON → 400 statt 500. 1 neuer Test |
| H3 | HIGH | process.env in SolanaProvider statt clientEnv (Pattern-Verletzung aus Story 1-3 Review H2) | `clientEnv.NEXT_PUBLIC_SOLANA_RPC_URL` statt `process.env` |
| M1 | MEDIUM | ChainFilter activeChain State hat keine Auswirkung — Filter rein visuell | `onChainChange` Callback-Prop ergaenzt. Filterlogik kommt mit Epic 2 Daten |
| M2 | MEDIUM | Provider-Hierarchy Dokumentation widerspricht Implementation | Dev Notes korrigiert: Web3Provider outermost, ThemeProvider innermost |
| M3 | MEDIUM | drizzle/meta/ Dateien fehlen in File List (wiederholt aus Story 1-2 M3) | `drizzle/meta/_journal.json` und `drizzle/meta/0003_snapshot.json` ergaenzt |

**Nicht gefixt (3 Low-Issues — akzeptabel fuer MVP):**

| # | Severity | Issue | Grund |
|---|----------|-------|-------|
| L1 | LOW | generateLinkingMessage() ist toter Code — wird serverseitig nie aufgerufen | Kann spaeter fuer server-side Message-Template-Validation genutzt werden |
| L2 | LOW | Emoji-Spinner statt CSS/SVG Loading Indicator | Wird in spaeterer UI-Polish Story ersetzt |
| L3 | LOW | truncateAddress() fuer UUID-userId — semantisch falsch | Funktioniert fuer Logging-Zwecke, Rename zu truncateId() bei nächstem Refactoring |

### Verification

- 17/17 Vitest Test-Dateien bestanden
- 92 Tests total (3 neue: nonce-missing, nonce-mismatch, invalid-atom-json)
- 0 Regressionen

### File List

**Neue Dateien:**
- src/infrastructure/db/repositories/linked-wallet-repository.ts
- src/infrastructure/db/repositories/linked-wallet-repository.test.ts
- src/lib/auth/wallet-verification.ts
- src/lib/auth/wallet-verification.test.ts
- src/providers/solana-provider.tsx
- src/app/api/wallets/link/route.ts
- src/app/api/wallets/link/route.test.ts
- src/app/api/wallets/route.ts
- src/app/api/wallets/route.test.ts
- src/app/(dashboard)/settings/page.tsx
- src/components/settings/AddWalletDialog.tsx
- src/components/dashboard/ChainFilter.tsx
- src/lib/hooks/use-linked-wallets.ts
- src/types/keplr.d.ts
- drizzle/0003_faithful_natasha_romanoff.sql (Migration)
- drizzle/meta/_journal.json (Migration Journal aktualisiert)
- drizzle/meta/0003_snapshot.json (Migration Snapshot)

**Geänderte Dateien:**
- src/infrastructure/db/schema.ts (linked_wallets Tabelle)
- src/infrastructure/db/schema.test.ts (2 neue Tests)
- src/lib/env.ts (NEXT_PUBLIC_SOLANA_RPC_URL)
- .env.example (Solana RPC URL)
- src/app/layout.tsx (SolanaProvider einbinden)
- src/app/(dashboard)/page.tsx (ChainFilter eingebunden)
- src/middleware.ts (/settings Route)
- package.json (Solana + Keplr Packages)
- package-lock.json (neue Abhängigkeiten)
- _bmad-output/implementation-artifacts/sprint-status.yaml (Status-Update)
