# Story 1.2: ETH-Wallet-Authentifizierung (SIWE)

Status: done

<!-- Note: Validation is optional. Run validate-create-story for quality check before dev-story. -->

## Story

As a User mit einer Ethereum-Wallet,
I want mich per Wallet-Signatur authentifizieren können,
So that ich sicher und ohne Passwort auf mein StakeTrack AI Dashboard zugreifen kann.

## Acceptance Criteria

1. **Given** ein nicht-authentifizierter User auf der Signin-Page **When** er auf "Connect Wallet" klickt **Then** öffnet sich das RainbowKit-Modal mit verfügbaren ETH-Wallets (MetaMask, WalletConnect, etc.) **And** das Modal ist im Dark Theme gestaltet (konsistent mit StakeTrack AI Design)

2. **Given** ein User hat eine ETH-Wallet im RainbowKit-Modal ausgewählt **When** die SIWE-Signatur-Anfrage erscheint **Then** enthält die Nachricht eine Server-generierte Nonce (Replay-Schutz, NFR8) **And** die Nachricht erklärt klar "Read-Only-Zugang zu StakeTrack AI"

3. **Given** ein User signiert die SIWE-Nachricht erfolgreich **When** die Signatur serverseitig verifiziert wird **Then** wird ein User-Eintrag in der `users`-Tabelle erstellt (falls neu) oder aktualisiert (falls bestehend) **And** eine JWT-Session wird erstellt (NextAuth) mit angemessener Expiry-Zeit (NFR16) **And** der User wird zum Dashboard weitergeleitet **And** die Wallet-Adresse wird NICHT vollständig in Server-Logs geschrieben (nur gekürzt: 0x1234...5678, NFR15)

4. **Given** ein User lehnt die Signatur ab **When** die Ablehnung erkannt wird **Then** zeigt die UI einen sachlichen Hinweis: "Signatur nötig für Read-Only-Zugang" **And** der User kann es erneut versuchen

5. **Given** ein bereits authentifizierter User **When** er die App öffnet **Then** wird die bestehende JWT-Session genutzt (kein erneuter Wallet-Connect nötig)

6. **Given** die Wallet-Verbindung schlägt fehl (Timeout, Netzwerkfehler) **When** der Fehler erkannt wird **Then** zeigt die UI "Verbindung fehlgeschlagen. Nochmal versuchen?" mit Retry-Button

## Tasks / Subtasks

- [x] Task 1: Auth-Pakete installieren (AC: alle)
  - [x] 1.1: `npm install next-auth@4 @rainbow-me/rainbowkit@^2.2.10 @rainbow-me/rainbowkit-siwe-next-auth@^0.5.0 wagmi@^2.14 viem@^2.46 @tanstack/react-query@^5` (--legacy-peer-deps wegen React 19)
  - [x] 1.2: Env-Vars in `src/lib/env.ts` ergänzen falls nötig (NEXTAUTH_SECRET, NEXTAUTH_URL, NEXT_PUBLIC_WALLETCONNECT_PROJECT_ID bereits vorhanden)

- [x] Task 2: DB-Schema — `users`-Tabelle erstellen (AC: #3)
  - [x] 2.1: `users`-Tabelle in `src/infrastructure/db/schema.ts` definieren: id (uuid, PK), wallet_address (varchar, unique, nullable), email (varchar, nullable), tier (enum: free/pro, default: 'free'), created_at (timestamp), updated_at (timestamp)
  - [x] 2.2: `drizzle-kit generate` für Migration-SQL ausführen
  - [x] 2.3: Migration auf NeonDB anwenden (`drizzle-kit migrate`)
  - [x] 2.4: Schema-Tests in `schema.test.ts` erweitern

- [x] Task 3: NextAuth Konfiguration (AC: #2, #3, #5)
  - [x] 3.1: `src/lib/auth/auth-options.ts` erstellen: NextAuth Config mit CredentialsProvider für SIWE
  - [x] 3.2: SIWE-Nonce-Endpoint implementieren (Server-generierte Nonce für Replay-Schutz)
  - [x] 3.3: SIWE-Verify-Logik: Signatur serverseitig verifizieren via `viem/siwe` (`verifySiweMessage`)
  - [x] 3.4: JWT-Callback: `wallet_address`, `userId`, `tier` in Token aufnehmen
  - [x] 3.5: Session-Callback: Token-Daten in Session exponieren
  - [x] 3.6: JWT Expiry-Zeit konfigurieren (z.B. 7 Tage, NFR16)
  - [x] 3.7: `src/lib/auth/session.ts` — `getServerSession` Wrapper Helper erstellen

- [x] Task 4: NextAuth API Route (AC: #3, #5)
  - [x] 4.1: `src/app/api/auth/[...nextauth]/route.ts` erstellen
  - [x] 4.2: GET und POST Handler exportieren

- [x] Task 5: wagmi + RainbowKit Provider Setup (AC: #1)
  - [x] 5.1: `src/providers/web3-provider.tsx` erstellen: wagmiConfig mit mainnet Chain, WalletConnect projectId, RainbowKit-kompatible Konfiguration
  - [x] 5.2: `src/providers/query-provider.tsx` erstellen: TanStack Query Client
  - [x] 5.3: `src/providers/auth-provider.tsx` erstellen: `RainbowKitSiweNextAuthProvider` + `SessionProvider`
  - [x] 5.4: `src/app/layout.tsx` aktualisieren: Provider-Hierarchy einbauen (WagmiProvider → QueryProvider → SessionProvider → RainbowKitSiweNextAuthProvider → RainbowKitProvider → ThemeProvider → children)
  - [x] 5.5: RainbowKit Dark Theme konfigurieren (Custom Theme mit StakeTrack AI Farben: bg-base, brand-primary #06B6D4)

- [x] Task 6: User Repository (AC: #3)
  - [x] 6.1: `src/infrastructure/db/repositories/user-repository.ts` erstellen: `findByWalletAddress()`, `createUser()`, `updateUser()`
  - [x] 6.2: User-Upsert-Logik: Bei erfolgreichem SIWE → User erstellen (falls neu) oder `updated_at` aktualisieren

- [x] Task 7: Signin-Page UI (AC: #1, #4, #6)
  - [x] 7.1: `src/app/auth/signin/page.tsx` erstellen: Signin-Page mit RainbowKit ConnectButton
  - [x] 7.2: Dark-Theme-konformes Layout (bg-base, brand-primary)
  - [x] 7.3: Error-Handling-UI: Signatur abgelehnt → sachlicher Hinweis, Verbindungsfehler → Retry-Button
  - [x] 7.4: Redirect nach erfolgreichem Login → `/dashboard`

- [x] Task 8: Header ConnectButton Integration (AC: #1, #5)
  - [x] 8.1: `src/components/auth/ConnectWalletButton.tsx` erstellen: RainbowKit `<ConnectButton>` Wrapper mit Custom-Rendering (gekürzte Adresse, Chain-Badge)
  - [x] 8.2: `src/components/layout/Header.tsx` aktualisieren: Statischen "Connect Wallet"-Text durch `<ConnectWalletButton />` ersetzen
  - [x] 8.3: Authentifizierter Zustand: Gekürzte Wallet-Adresse anzeigen (0x1234...5678)
  - [x] 8.4: Nicht-authentifizierter Zustand: "Connect Wallet" Button

- [x] Task 9: Auth Middleware / Route Protection (AC: #5)
  - [x] 9.1: `src/middleware.ts` erstellen oder erweitern: Unauthentifizierte User von `/dashboard/*` zu `/auth/signin` redirecten
  - [x] 9.2: Authentifizierte User von `/auth/signin` zu `/dashboard` redirecten

- [x] Task 10: Logging-Compliance (AC: #3, NFR15)
  - [x] 10.1: `src/lib/utils/format.ts` erstellen: `truncateAddress(address: string)` → `0x1234...5678`
  - [x] 10.2: In allen Auth-Logs `truncateAddress()` verwenden — KEINE vollständigen Wallet-Adressen in Logs

- [x] Task 11: Tests (AC: alle)
  - [x] 11.1: `src/lib/auth/auth-options.test.ts` — SIWE Verify-Logik, JWT-Callbacks, Session-Callbacks
  - [x] 11.2: `src/infrastructure/db/repositories/user-repository.test.ts` — User Upsert-Logik
  - [x] 11.3: `src/components/auth/ConnectWalletButton.test.tsx` — Render-Tests
  - [x] 11.4: `src/lib/utils/format.test.ts` — truncateAddress Tests
  - [x] 11.5: Dev-Server Smoke-Test: Login-Flow manuell verifizieren

## Dev Notes

### Kritische Architektur-Entscheidungen

- **next-auth v4 (NICHT v5/Auth.js):** Das RainbowKit SIWE-Paket (`@rainbow-me/rainbowkit-siwe-next-auth@0.5.0`) ist EXPLIZIT inkompatibel mit next-auth v5. next-auth v4.24.13 ist die letzte stabile Version. Auth.js hat sich mit Better Auth zusammengeschlossen (September 2025) — next-auth v4 erhält nur noch Security-Patches. **Post-MVP evaluieren: Migration zu Better Auth + RainbowKit Custom Auth Adapter.**
- **wagmi v2 (NICHT v3):** RainbowKit 2.2.10 hat eine Peer-Dependency auf `wagmi: "^2.9.0"`. wagmi v3 wird NICHT unterstützt (offene GitHub Discussion #2575, kein Timeline). wagmi MUSS auf v2.14.x gepinnt werden.
- **`viem/siwe` statt standalone `siwe` Paket:** Das `siwe` NPM-Paket (v3.0.0) ist seit ~1 Jahr nicht aktualisiert. `viem` bündelt SIWE nativ (`createSiweMessage`, `verifySiweMessage` aus `viem/siwe`). Das `rainbowkit-siwe-next-auth` Paket nutzt intern bereits `viem/siwe`.
- **React 19 + RainbowKit:** RainbowKit's Peer-Dep sagt `react: ">=18"`. Das offizielle Scaffold-Template wurde auf React 19 aktualisiert. Funktioniert, aber npm warnt — `--legacy-peer-deps` verwenden (wie bereits bei Tremor in Story 1.1).
- **`@tanstack/react-query` als neue Dependency:** Wird von wagmi/RainbowKit als Peer-Dep benötigt. Wird später auch für Client-Side Data Layer (Dashboard-Daten, Chain-Filter) genutzt — kein zusätzlicher Bundle-Overhead.
- **Provider-Reihenfolge KRITISCH:** `WagmiProvider` → `QueryClientProvider` → `SessionProvider` → `RainbowKitSiweNextAuthProvider` → `RainbowKitProvider` → ThemeProvider. Falsche Reihenfolge führt zu Runtime-Errors.
- **Alle RainbowKit-Komponenten sind Client Components:** `'use client'` Directive PFLICHT auf jeder Datei die RainbowKit-Hooks oder -Komponenten nutzt. RSC-Default wird beibehalten, nur Auth-UI ist Client-Side.

### DB-Schema: `users`-Tabelle

```sql
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  wallet_address VARCHAR(42) UNIQUE,
  email VARCHAR(255) UNIQUE,
  tier VARCHAR(10) NOT NULL DEFAULT 'free' CHECK (tier IN ('free', 'pro')),
  created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_users_wallet_address ON users (wallet_address) WHERE wallet_address IS NOT NULL;
CREATE INDEX idx_users_email ON users (email) WHERE email IS NOT NULL;
```

**Drizzle-Schema (TypeScript):**
```typescript
// src/infrastructure/db/schema.ts
import { pgTable, uuid, varchar, timestamp, index } from "drizzle-orm/pg-core";

export const users = pgTable("users", {
  id: uuid("id").primaryKey().defaultRandom(),
  walletAddress: varchar("wallet_address", { length: 42 }).unique(),
  email: varchar("email", { length: 255 }).unique(),
  tier: varchar("tier", { length: 10 }).notNull().default("free"),
  createdAt: timestamp("created_at", { withTimezone: true }).notNull().defaultNow(),
  updatedAt: timestamp("updated_at", { withTimezone: true }).notNull().defaultNow(),
}, (table) => [
  index("idx_users_wallet_address").on(table.walletAddress),
  index("idx_users_email").on(table.email),
]);
```

### RainbowKit Dark Theme Konfiguration

```typescript
import { darkTheme } from "@rainbow-me/rainbowkit";

const stakeTrackTheme = darkTheme({
  accentColor: "#06B6D4",        // brand-primary (Teal)
  accentColorForeground: "#0A0A0F", // bg-base
  borderRadius: "medium",
  fontStack: "system",
});

// Override für StakeTrack AI Hintergrund
stakeTrackTheme.colors.modalBackground = "#12121A";   // bg-surface
stakeTrackTheme.colors.profileForeground = "#1A1A2E"; // bg-surface-raised
```

### SIWE Message Format

```
staketrack.ai wants you to sign in with your Ethereum account:
0x1234...5678

Read-Only-Zugang zu StakeTrack AI — deine Wallet-Keys werden NIEMALS angefordert oder gespeichert.

URI: https://staketrack.ai
Version: 1
Chain ID: 1
Nonce: {server-generated-nonce}
Issued At: {iso-timestamp}
```

### JWT Session Claims

```typescript
// Token enthält:
{
  sub: string,          // wallet_address (lowercase)
  userId: string,       // UUID aus users-Tabelle
  tier: "free" | "pro", // User-Tier für Client-Side Feature-Gating
  iat: number,
  exp: number           // 7 Tage Expiry
}
```

### Paketversionen (Stand: 2026-02-20)

| Paket | Version | Hinweis |
|---|---|---|
| `next-auth` | 4.24.13 | Letzte stabile v4, Security-Patches only |
| `@rainbow-me/rainbowkit` | 2.2.10 | Letzte stabile, Dez 2024 |
| `@rainbow-me/rainbowkit-siwe-next-auth` | 0.5.0 | Maintenance Mode, funktioniert mit next-auth v4 |
| `wagmi` | ~2.14.x | MUSS v2 sein (RainbowKit Peer-Dep) |
| `viem` | ~2.46.x | Inkludiert `viem/siwe` |
| `@tanstack/react-query` | ~5.x | Peer-Dep von wagmi/RainbowKit |

### Learnings aus Story 1-1

- **`--legacy-peer-deps` nötig:** Tremor 3.18.7 hatte React 18 Peer-Dep. RainbowKit wird dasselbe Problem haben → `--legacy-peer-deps` bei Installation verwenden.
- **Tailwind v4:** Kein `tailwind.config.ts` — alle Custom Tokens in `globals.css` via `@theme`. RainbowKit-Theming erfolgt programmatisch (JS), nicht via Tailwind Config.
- **Drizzle `neon-http` Modus:** DB-Client nutzt HTTP-basierten Treiber — geeignet für einzelne Queries in Serverless Functions. Für Transaktionen (z.B. User-Upsert) reicht eine einzelne INSERT ... ON CONFLICT Query.
- **Next.js 15.5.12 (LTS):** App Router, Turbopack Dev. `src/` Verzeichnis, `@/` Path-Alias konfiguriert.
- **Biome 2.2.0:** Linter + Formatter, `linter.domains.next` und `react` aktiv. Alle neuen Dateien müssen Biome-Check bestehen.
- **Breakpoints:** Mobile <768px, Tablet 768-1024px, Desktop >1024px. `lg:` Prefix für Desktop. Sidebar nur Desktop, BottomNav nur Mobile.

### Bestehende Infrastruktur (aus Story 1-1)

| Komponente | Datei | Status |
|---|---|---|
| DB Client | `src/infrastructure/db/index.ts` | ✅ Existiert (Drizzle + neon-http) |
| DB Schema | `src/infrastructure/db/schema.ts` | ⚠️ Leer (`export {}`) → MUSS erweitert werden |
| Env Validation | `src/lib/env.ts` | ✅ NEXTAUTH_SECRET, NEXTAUTH_URL, WALLETCONNECT_PROJECT_ID bereits definiert |
| Root Layout | `src/app/layout.tsx` | ✅ ThemeProvider → MUSS um Auth/Web3 Provider erweitert werden |
| Dashboard Layout | `src/app/(dashboard)/layout.tsx` | ✅ Header + Sidebar + BottomNav |
| Header | `src/components/layout/Header.tsx` | ⚠️ Statischer "Connect Wallet" Text → MUSS durch ConnectButton ersetzt werden |
| ThemeProvider | `src/providers/theme-provider.tsx` | ✅ Dark/Light Toggle funktioniert |
| Auth Dir | `src/lib/auth/` | ⚠️ Leer (.gitkeep) → MUSS erstellt werden |
| Auth Components | `src/components/auth/` | ⚠️ Leer (.gitkeep) → MUSS erstellt werden |
| Middleware | `src/middleware.ts` | ❌ Existiert nicht → MUSS erstellt werden |
| Utils/Format | `src/lib/utils/format.ts` | ❌ Existiert nicht → MUSS erstellt werden |

### Project Structure Notes

- Alle neuen Dateien folgen der DDD-Struktur aus dem Architecture Document
- Auth-Logik (NextAuth Config, SIWE Verify) gehört in `src/lib/auth/` (Shared Utilities Layer)
- User Repository gehört in `src/infrastructure/db/repositories/` (Infrastructure Layer)
- ConnectWalletButton gehört in `src/components/auth/` (UI Layer)
- Web3/Auth Provider gehören in `src/providers/` (Provider Layer)
- API Route gehört in `src/app/api/auth/[...nextauth]/` (App Layer)
- Domain Layer (`src/domain/`) wird in dieser Story NICHT berührt (kein Business Logic nötig)

### References

- [Source: _bmad-output/planning-artifacts/epics.md#Story 1.2]
- [Source: _bmad-output/planning-artifacts/architecture.md#Authentication & Security]
- [Source: _bmad-output/planning-artifacts/architecture.md#Core Architectural Decisions]
- [Source: _bmad-output/planning-artifacts/architecture.md#Implementation Patterns & Consistency Rules]
- [Source: _bmad-output/planning-artifacts/architecture.md#Complete Project Directory Structure]
- [Source: _bmad-output/planning-artifacts/prd.md#FR1 — ETH-Wallet-Connect + SIWE-Auth]
- [Source: _bmad-output/planning-artifacts/prd.md#NFR8 — SIWE serverseitig verifiziert]
- [Source: _bmad-output/planning-artifacts/prd.md#NFR15 — Keine PII in Logs]
- [Source: _bmad-output/planning-artifacts/prd.md#NFR16 — JWT-Sessions]
- [Source: _bmad-output/planning-artifacts/ux-design-specification.md#Flow 2: Wallet-Connect → First Insight]
- [Source: _bmad-output/planning-artifacts/ux-design-specification.md#Customization Strategy — Wallet-Connect-Button]
- [Source: _bmad-output/planning-artifacts/ux-design-specification.md#Dark-Mode-Farbsystem]
- [Source: _bmad-output/project-context.md#Authentication Rules]
- [Source: _bmad-output/project-context.md#Auth Stack]
- [Source: _bmad-output/project-context.md#Security Rules]
- [Source: _bmad-output/implementation-artifacts/1-1-projekt-initialisierung-infrastruktur-setup.md — Learnings]

## Dev Agent Record

### Agent Model Used

Claude Opus 4.6 (claude-opus-4-6)

### Debug Log References

- Biome auto-fixed 10 files (import sorting, formatting)
- TypeScript strict mode required NextAuth type augmentation via `src/types/next-auth.d.ts`
- RainbowKit SIWE uses NextAuth's `getCsrfToken()` as nonce — kein separater Nonce-Endpoint nötig
- QueryClient und Web3Provider in einer Datei kombiniert (statt separater query-provider.tsx), da wagmi/RainbowKit diese als integrierte Provider-Chain benötigt

### Completion Notes List

- Task 1: Auth-Pakete installiert (next-auth@4, rainbowkit@2.2.10, wagmi@2.19.5, viem@2.46, react-query@5) mit --legacy-peer-deps für React 19
- Task 2: users-Tabelle in Drizzle Schema definiert (UUID PK, wallet_address, email, tier, timestamps), Migration generiert und auf NeonDB angewendet
- Task 3: NextAuth konfiguriert mit CredentialsProvider für SIWE, JWT-Callbacks mit userId/tier/walletAddress, 7-Tage-Session (NFR16), serverseitige SIWE-Verifikation via viem/siwe
- Task 4: NextAuth API Route unter /api/auth/[...nextauth] mit GET/POST Handler
- Task 5: wagmi + RainbowKit Provider mit StakeTrack AI Dark Theme, Provider-Hierarchy korrekt eingebaut in Root Layout
- Task 6: User Repository mit findByWalletAddress, createUser, updateUser, upsertByWalletAddress — alle lowercase-normalisiert
- Task 7: Signin-Page mit RainbowKit ConnectButton, Dark-Theme-Layout, Read-Only-Hinweis, Auto-Redirect bei Auth
- Task 8: ConnectWalletButton Custom-Wrapper mit gekürzter Adresse und Chain-Badge, Header aktualisiert
- Task 9: Middleware für Route Protection — /dashboard/* protected, /auth/signin redirect für Auth-Users
- Task 10: truncateAddress Utility für Logging-Compliance (NFR15)
- Task 11: 16 neue Tests (auth-options: 5, user-repository: 4, ConnectWalletButton: 1, format: 4, schema: 2) — alle 30 Tests bestanden
- Subtask 11.5 (manueller Smoke-Test) verbleibt beim User

### Implementation Plan

- SIWE-Auth-Flow: User klickt Connect Wallet → RainbowKit Modal → Wallet-Auswahl → SIWE-Signatur → Server-Verifikation → User-Upsert → JWT-Session
- Provider-Hierarchy: WagmiProvider → QueryClientProvider → SessionProvider → RainbowKitSiweNextAuthProvider → RainbowKitProvider → ThemeProvider
- Nonce-Replay-Schutz: RainbowKit SIWE nutzt NextAuth CSRF Token als Nonce (NFR8)
- Keine PII in Logs: truncateAddress() für alle Wallet-Adressen in Logging (NFR15)

### File List

**Neue Dateien:**
- src/infrastructure/db/schema.ts (modifiziert: users-Tabelle hinzugefügt)
- src/infrastructure/db/schema.test.ts (modifiziert: users-Tabelle-Tests)
- src/infrastructure/db/repositories/user-repository.ts (neu)
- src/infrastructure/db/repositories/user-repository.test.ts (neu)
- src/lib/auth/auth-options.ts (neu)
- src/lib/auth/auth-options.test.ts (neu)
- src/lib/auth/session.ts (neu)
- src/lib/utils/format.ts (neu)
- src/lib/utils/format.test.ts (neu)
- src/app/api/auth/[...nextauth]/route.ts (neu)
- src/app/auth/signin/page.tsx (neu)
- src/providers/web3-provider.tsx (neu)
- src/providers/auth-provider.tsx (neu)
- src/components/auth/ConnectWalletButton.tsx (neu)
- src/components/auth/ConnectWalletButton.test.tsx (neu)
- src/types/next-auth.d.ts (neu)
- src/middleware.ts (neu)
- drizzle/0000_gigantic_nitro.sql (neu: Migration)
- drizzle/meta/0000_snapshot.json (neu: Migration Metadata)
- drizzle/meta/_journal.json (neu: Migration Journal)
- drizzle/0001_eminent_cobalt_man.sql (neu: CHECK Constraint Migration)

**Modifizierte Dateien:**
- src/app/layout.tsx (Provider-Hierarchy hinzugefügt)
- src/components/layout/Header.tsx (ConnectWalletButton Integration)
- src/components/layout/Header.test.tsx (Mock für ConnectWalletButton)
- package.json (Auth-Dependencies hinzugefügt)
- package-lock.json (Lock-File aktualisiert)

## Change Log

- 2026-02-20: Story 1-2 implementiert — ETH-Wallet-Authentifizierung via SIWE mit RainbowKit, NextAuth v4 JWT-Sessions, users-Tabelle, Route Protection Middleware, 30 Tests bestanden
- 2026-02-20: Code Review (AI) — 4 HIGH, 3 MEDIUM, 2 LOW Issues gefunden. Fixes angewendet:
  - H1: Auth-Logging mit truncateAddress() hinzugefügt (NFR15-Compliance)
  - H2: Signin-Page Error-Handling UI hinzugefügt (AC4 Signatur-Ablehnung, AC6 Verbindungsfehler)
  - H3: Race Condition in upsertUserByWalletAddress gefixt → INSERT...ON CONFLICT
  - H4: Redirect-Ziel korrigiert: /dashboard statt /
  - M1: CHECK Constraint auf tier-Spalte hinzugefügt (Drizzle Schema + Migration 0001 generiert und angewendet)
  - M2: QueryClient in useState verschoben (SSR Data Leakage Prevention)
  - M3: drizzle/meta/_journal.json in File List ergänzt
  - M1 Migration 0001_eminent_cobalt_man.sql generiert und auf NeonDB angewendet
  - Smoke-Test bestanden: chainsightsone.eth erfolgreich verbunden
  - Status: done
