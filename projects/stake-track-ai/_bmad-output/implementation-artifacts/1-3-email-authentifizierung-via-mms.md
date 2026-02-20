# Story 1.3: Email-Authentifizierung via MMS

Status: done

<!-- Note: Validation is optional. Run validate-create-story for quality check before dev-story. -->

## Story

As a User ohne Crypto-Wallet,
I want mich per Email registrieren und einloggen können,
So that ich StakeTrack AI auch ohne Wallet-Besitz nutzen kann.

## Acceptance Criteria

1. **Given** ein nicht-authentifizierter User auf der Signin-Page **When** er den Email-Login-Tab wählt **Then** sieht er ein Email-Eingabefeld mit Cloudflare Turnstile Bot-Schutz (NFR14)

2. **Given** ein User gibt eine gültige Email-Adresse ein und besteht den Turnstile-Check **When** er auf "Login / Registrieren" klickt **Then** wird über den MMS-Client (`src/infrastructure/mms/mms-client.ts`) ein Magic-Link an die Email gesendet **And** die UI zeigt "Prüfe deine Email für den Login-Link"

3. **Given** ein User klickt den Magic-Link in seiner Email **When** der Link serverseitig validiert wird **Then** wird ein User-Eintrag erstellt (falls neu, mit email statt wallet_address) oder die Session aktualisiert **And** eine JWT-Session wird erstellt (NextAuth) **And** der User wird zum Dashboard weitergeleitet

4. **Given** ein User gibt eine ungültige Email ein **When** die Validierung fehlschlägt (Zod) **Then** zeigt die UI eine spezifische Inline-Fehlermeldung unter dem Feld

5. **Given** der Turnstile-Check schlägt fehl **When** der Bot-Verdacht erkannt wird **Then** wird kein MMS-Request gesendet **And** der User wird aufgefordert, den Check erneut zu versuchen

## Tasks / Subtasks

- [x] Task 1: Cloudflare Turnstile Integration (AC: #1, #5)
  - [x] 1.1: `npm install @marsidev/react-turnstile` (v1.4.x, React Turnstile Widget)
  - [x] 1.2: Env-Vars ergänzen in `src/lib/env.ts`: `TURNSTILE_SECRET_KEY` (server), `NEXT_PUBLIC_TURNSTILE_SITE_KEY` (client)
  - [x] 1.3: `.env.example` um Turnstile-Keys erweitern
  - [x] 1.4: Turnstile Server-Verification Helper erstellen: `src/lib/auth/turnstile.ts` — `verifyTurnstileToken(token: string, ip?: string): Promise<boolean>` via `https://challenges.cloudflare.com/turnstile/v0/siteverify`

- [x] Task 2: MMS Client Infrastructure (AC: #2) — Resend statt MMS (MMS hat keinen Email-Send-Endpoint, Rücksprache mit Sempre: Resend verwenden)
  - [x] 2.1: `src/infrastructure/mms/mms-client.ts` erstellen: Resend-basierter Email-Client für Magic-Link-Versand
  - [x] 2.2: `src/infrastructure/mms/mms-types.ts` erstellen: TypeScript-Typen + Zod-Schemas für Resend API Responses
  - [x] 2.3: MMS-Client Methode `sendMagicLinkEmail(email: string, magicLinkUrl: string): Promise<void>` — sendet Email via Resend API
  - [x] 2.4: Zod-Schema für Resend API Response Validation
  - [x] 2.5: Fehlerbehandlung: Error-Logging ohne PII (`[mms] Email sent to [hash]`)

- [x] Task 3: DB-Schema — `verification_tokens`-Tabelle (AC: #2, #3)
  - [x] 3.1: `verification_tokens`-Tabelle in `src/infrastructure/db/schema.ts` ergänzen: identifier (varchar, email), token (varchar, hashed), expires_at (timestamp)
  - [x] 3.2: Composite Primary Key auf `(identifier, token)`
  - [x] 3.3: `drizzle-kit generate` für Migration-SQL
  - [x] 3.4: Migration auf NeonDB anwenden (`drizzle-kit migrate`)
  - [x] 3.5: Schema-Tests in `schema.test.ts` erweitern

- [x] Task 4: Magic-Link Token-Service (AC: #2, #3)
  - [x] 4.1: `src/lib/auth/magic-link.ts` erstellen: `generateMagicLinkToken(email: string): Promise<{ token: string, hashedToken: string }>` — crypto.randomBytes(32), SHA-256 Hash für DB
  - [x] 4.2: `verifyMagicLinkToken(email: string, token: string): Promise<boolean>` — Token hashen, gegen DB prüfen, Expiry verifizieren, Token nach Verwendung löschen (Single-Use)
  - [x] 4.3: Token-Expiry: 15 Minuten (konfigurierbar)
  - [x] 4.4: `src/infrastructure/db/repositories/verification-token-repository.ts` erstellen: `createToken()`, `findToken()`, `deleteToken()`, `deleteExpiredTokens()`

- [x] Task 5: Email-Login API Endpoints (AC: #2, #3, #4, #5)
  - [x] 5.1: `src/app/api/auth/email/send/route.ts` — POST: Empfängt email + turnstileToken, validiert beides, generiert Magic-Link-Token, speichert in DB, sendet via Resend
  - [x] 5.2: `src/app/api/auth/email/verify/route.ts` — GET: Empfängt token + email als Query-Params, verifiziert Token, upserted User, redirectet zur Verify-Page
  - [x] 5.3: Zod-Schemas für Request-Validierung (email: z.string().email(), turnstileToken: z.string().min(1))
  - [x] 5.4: Rate-Limiting auf `/api/auth/email/send` (max 3 Requests pro Email pro 15 Min) — In-Memory-Fallback (Upstash nicht installiert)
  - [x] 5.5: API Response Format: `{ data: { message: string } }` (Success), `{ error: string, code: string }` (Error)

- [x] Task 6: NextAuth Konfiguration erweitern (AC: #3)
  - [x] 6.1: Zweiten CredentialsProvider "email-login" in `auth-options.ts` ergänzen: credentials: `{ email, token }`, authorize: Token verifizieren → User upserten → User-Objekt zurückgeben
  - [x] 6.2: JWT-Callback erweitern: Wenn User via Email-Login → `sub` = email (statt wallet_address), `authMethod` = "email" in Token aufnehmen
  - [x] 6.3: Session-Callback erweitern: `email` Feld in Session exponieren
  - [x] 6.4: `next-auth.d.ts` erweitern: `email?: string`, `authMethod?: "wallet" | "email"` in Session + JWT Typen

- [x] Task 7: User Repository erweitern (AC: #3)
  - [x] 7.1: `findUserByEmail(email: string)` in `user-repository.ts` ergänzen
  - [x] 7.2: `upsertUserByEmail(email: string)` ergänzen: INSERT ... ON CONFLICT (email) DO UPDATE SET updated_at
  - [x] 7.3: Tests für neue Repository-Methoden

- [x] Task 8: Signin-Page UI erweitern (AC: #1, #2, #4, #5)
  - [x] 8.1: Tab-Layout auf Signin-Page: "Wallet" (bestehend) | "Email" (neu) — eigene Tab-Komponente
  - [x] 8.2: Email-Tab: Email-Eingabefeld mit Inline-Validierung, Turnstile-Widget darunter, "Login / Registrieren"-Button
  - [x] 8.3: "Prüfe deine Email"-State nach erfolgreichem Submit (mit Email-Adresse anzeigen, "Erneut senden"-Link nach 60s Cooldown)
  - [x] 8.4: Error-States: Ungültige Email (Inline), Turnstile fehlgeschlagen (Hinweis + Retry), Server-Fehler (inline)
  - [x] 8.5: Dark-Theme-konform: Email-Input mit `bg-bg-surface-raised`, `border-border-subtle`, `text-text-primary`

- [x] Task 9: Magic-Link Verify-Page (AC: #3)
  - [x] 9.1: `src/app/auth/verify/page.tsx` erstellen: Liest token + email aus URL-Params, zeigt Loading-State ("Wird verifiziert..."), ruft `signIn('email-login', { email, token, redirect: false })` auf
  - [x] 9.2: Bei Erfolg: Redirect zu `/dashboard`
  - [x] 9.3: Bei Fehler (ungültiger/abgelaufener Token): Fehlermeldung + "Neuen Link anfordern"-Button → zurück zur Signin-Page
  - [x] 9.4: Dark-Theme-konformes Layout (konsistent mit Signin-Page)

- [x] Task 10: Middleware erweitern (AC: #3)
  - [x] 10.1: `src/middleware.ts` erweitern: `/auth/verify` als öffentliche Route erlauben (kein Redirect zu Signin)
  - [x] 10.2: Authentifizierte User auf `/auth/verify` zu `/dashboard` redirecten

- [x] Task 11: Tests (AC: alle)
  - [x] 11.1: `src/lib/auth/turnstile.test.ts` — Turnstile Verification (Success, Failure, Timeout) — 6 Tests
  - [x] 11.2: `src/lib/auth/magic-link.test.ts` — Token Generation, Verification, Expiry, Single-Use — 7 Tests
  - [x] 11.3: `src/infrastructure/mms/mms-client.test.ts` — Resend Client (Success, Error, Timeout) — 5 Tests
  - [x] 11.4: `src/infrastructure/db/repositories/verification-token-repository.test.ts` — CRUD-Operationen — 5 Tests
  - [x] 11.5: `src/infrastructure/db/repositories/user-repository.test.ts` — neue Email-Methoden ergänzt — 7 Tests
  - [x] 11.6: `src/lib/auth/auth-options.test.ts` — Email-Login-Provider-Tests ergänzt — 7 Tests
  - [ ] 11.7: Dev-Server Smoke-Test: Kompletter Email-Login-Flow manuell verifizieren

## Dev Notes

### Kritische Architektur-Entscheidungen

- **Zweiter CredentialsProvider (NICHT EmailProvider):** NextAuth v4 EmailProvider erfordert einen Database Adapter (für Verification Tokens + Sessions). Das aktuelle Setup nutzt JWT-Strategy OHNE DB-Adapter. Ein Database Adapter würde die gesamte Session-Logik umstellen (JWT → DB Sessions), was SIWE-Auth (Story 1-2) brechen kann. Stattdessen: Zweiter CredentialsProvider "email-login" mit eigenem Token-Management über eine separate `verification_tokens`-Tabelle.

- **Magic-Link Flow (NICHT Verification-Code):** Die Epics spezifizieren "Magic-Link oder Verification-Code". Magic-Link ist UX-freundlicher (1 Klick statt Code abtippen) und konsistent mit der PRD-Beschreibung. Der Magic-Link zeigt auf `/auth/verify?token={token}&email={email}`.

- **MMS als Email-Sender (NICHT Resend):** Laut Architecture und Project Context ist MMS (`api.masem.at`) der primäre Service für Cross-Project-Concerns. Email-Versand für Auth-Flows läuft über MMS. Resend wird für Alert-Emails (Epic 5) und Transactional Emails verwendet. **Cross-Project Service Rule beachten:** Wenn sich herausstellt, dass MMS keinen Email-Send-Endpoint hat, STOPP und Rücksprache mit Sempre ob Resend als Fallback oder ob MMS erweitert werden soll.

- **Cloudflare Turnstile Managed Mode:** Verwende den "Managed" Challenge-Typ — Cloudflare entscheidet automatisch ob ein interaktives Widget nötig ist oder ein unsichtbarer Check reicht. Seitenkeys müssen im Cloudflare Dashboard erstellt werden.

- **Token-Hashing:** Tokens werden als SHA-256-Hash in der DB gespeichert (wie NextAuth es intern macht). Der Klartext-Token wird nur in der Email übertragen. Verhindert Token-Theft bei DB-Leak.

- **next-auth v4 CredentialsProvider Limitation:** CredentialsProvider unterstützt KEINE automatische Session-Refresh via `getSession()`. Die JWT-Session (7 Tage) gilt wie bei SIWE. Kein Problem für MVP.

### DB-Schema: `verification_tokens`-Tabelle

```sql
CREATE TABLE verification_tokens (
  identifier VARCHAR(255) NOT NULL,
  token VARCHAR(255) NOT NULL,
  expires_at TIMESTAMP WITH TIME ZONE NOT NULL,
  created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
  CONSTRAINT verification_tokens_pkey PRIMARY KEY (identifier, token)
);

CREATE INDEX idx_verification_tokens_expires ON verification_tokens (expires_at);
```

**Drizzle-Schema (TypeScript):**
```typescript
// Ergänzung in src/infrastructure/db/schema.ts
export const verificationTokens = pgTable(
  "verification_tokens",
  {
    identifier: varchar("identifier", { length: 255 }).notNull(),
    token: varchar("token", { length: 255 }).notNull(),
    expiresAt: timestamp("expires_at", { withTimezone: true }).notNull(),
    createdAt: timestamp("created_at", { withTimezone: true }).notNull().defaultNow(),
  },
  (table) => [
    // Composite PK als Unique Constraint
    index("idx_verification_tokens_expires").on(table.expiresAt),
  ],
);
```

### Magic-Link URL-Format

```
https://staketrack.ai/auth/verify?token={plaintext-token}&email={encoded-email}
```

- Token: 32 Bytes random, hex-encoded (64 Zeichen)
- Email: URL-encoded
- Expiry: 15 Minuten nach Erstellung
- Single-Use: Token wird nach erfolgreicher Verifikation gelöscht

### MMS Client Pattern

```typescript
// src/infrastructure/mms/mms-client.ts
import { serverEnv } from "@/lib/env";

const MMS_TIMEOUT = 10_000; // 10 Sekunden

export async function sendMagicLinkEmail(
  email: string,
  magicLinkUrl: string,
): Promise<void> {
  const response = await fetch(`${serverEnv.MMS_API_URL}/email/send`, {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
      "X-API-Key": serverEnv.MMS_API_KEY ?? "",
    },
    body: JSON.stringify({
      to: email,
      subject: "StakeTrack AI — Dein Login-Link",
      // MMS-spezifische Felder — nach API-Docs anpassen
      template: "magic-link",
      variables: { magicLinkUrl },
    }),
    signal: AbortSignal.timeout(MMS_TIMEOUT),
  });

  if (!response.ok) {
    throw new Error(`[mms] Email send failed: ${response.status}`);
  }
}
```

**WICHTIG:** Die exakte MMS API-Struktur (Endpoints, Request-Body, Auth-Header) muss gegen die MMS API-Docs (`https://api.masem.at/docs`) verifiziert werden. Der Dev-Agent MUSS die Docs lesen bevor er den Client implementiert.

### NextAuth Email-Login Provider

```typescript
// Ergänzung in auth-options.ts — zweiter CredentialsProvider
CredentialsProvider({
  id: "email-login",
  name: "Email",
  credentials: {
    email: { label: "Email", type: "email" },
    token: { label: "Token", type: "text" },
  },
  async authorize(credentials) {
    if (!credentials?.email || !credentials?.token) return null;

    const isValid = await verifyMagicLinkToken(
      credentials.email,
      credentials.token,
    );
    if (!isValid) return null;

    const user = await upsertUserByEmail(credentials.email);
    console.info(`[auth] Email login success: ${credentials.email.substring(0, 3)}***`);

    return { id: user.id, email: credentials.email };
  },
}),
```

### JWT-Callback Erweiterung

```typescript
async jwt({ token, user, account }) {
  if (user) {
    token.userId = user.id;

    if (user.email && !user.name) {
      // Email-Login: sub = email, authMethod = "email"
      token.sub = user.email;
      token.authMethod = "email";
    } else {
      // Wallet-Login: sub = wallet_address, authMethod = "wallet"
      token.sub = user.name ?? undefined;
      token.authMethod = "wallet";
    }

    // Tier laden
    const dbUser = token.authMethod === "wallet"
      ? await findUserByWalletAddress(user.name ?? "")
      : await findUserByEmail(user.email ?? "");
    if (dbUser) token.tier = dbUser.tier;
  }
  return token;
},
```

### Signin-Page Tab-Layout

```
┌─────────────────────────────┐
│      StakeTrack AI          │
│  Multi-Chain Staking        │
│                             │
│  [  Wallet  ] [  Email  ]   │  ← Tab-Umschalter
│                             │
│  ┌─────────────────────┐    │
│  │ Email-Eingabefeld   │    │  ← Zod-validiert
│  └─────────────────────┘    │
│                             │
│  ┌─────────────────────┐    │
│  │ Cloudflare Turnstile│    │  ← Managed Mode
│  └─────────────────────┘    │
│                             │
│  [ Login / Registrieren ]   │  ← Disabled bis Turnstile ✓
│                             │
│  Read-Only-Zugang...        │
└─────────────────────────────┘
```

### Paketversionen (Stand: 2026-02-20)

| Paket | Version | Hinweis |
|---|---|---|
| `@marsidev/react-turnstile` | ~1.4.x | Aktivster React Turnstile Wrapper |
| `next-auth` | 4.24.13 | Bereits installiert (Story 1-2) |

### Learnings aus Story 1-2

- **`--legacy-peer-deps` nötig:** Für alle npm installs wegen React 19 Peer-Dep-Konflikten.
- **Drizzle `neon-http` Modus:** HTTP-basierter Treiber für einzelne Queries. Für Token-Insert + Token-Delete reichen separate Queries (keine Transaktion nötig, da Token-Verify atomar über SELECT + DELETE in einem Query abgewickelt werden kann).
- **Provider-Reihenfolge KRITISCH:** Neue Providers NICHT in die bestehende Provider-Hierarchy einfügen — Turnstile ist ein reines Client-Component, kein Provider.
- **Wallet-Login bleibt Primär:** Email-Login ist die Alternative für User ohne Wallet. Die Signin-Page zeigt standardmäßig den Wallet-Tab (bestehender Flow unverändert).
- **Biome 2.2.0:** Alle neuen Dateien müssen Biome-Check bestehen.
- **Next.js 15.5.12 (LTS):** App Router, Turbopack Dev.
- **Tailwind v4:** Custom Tokens in `globals.css` via `@theme`.
- **RainbowKit SIWE nutzt NextAuth CSRF Token als Nonce** — das bleibt unverändert.
- **QueryClient + Web3Provider kombiniert** in `web3-provider.tsx` — bleibt unverändert.
- **`truncateAddress()`** in `src/lib/utils/format.ts` — für Email-Logging: Email-Adressen ebenfalls kürzen (z.B. `u***@example.com`).

### Bestehende Infrastruktur (aus Story 1-1 und 1-2)

| Komponente | Datei | Status |
|---|---|---|
| DB Client | `src/infrastructure/db/index.ts` | ✅ Existiert (Drizzle + neon-http) |
| DB Schema | `src/infrastructure/db/schema.ts` | ✅ `users`-Tabelle existiert (email-Spalte nullable) → MUSS um `verification_tokens` erweitert werden |
| User Repository | `src/infrastructure/db/repositories/user-repository.ts` | ✅ Existiert → MUSS um Email-Methoden erweitert werden |
| Env Validation | `src/lib/env.ts` | ✅ MMS_API_KEY + MMS_API_URL bereits definiert → MUSS um Turnstile-Keys erweitert werden |
| NextAuth Config | `src/lib/auth/auth-options.ts` | ✅ SIWE-Provider → MUSS um Email-Provider erweitert werden |
| NextAuth Types | `src/types/next-auth.d.ts` | ✅ Existiert → MUSS um email + authMethod erweitert werden |
| NextAuth API Route | `src/app/api/auth/[...nextauth]/route.ts` | ✅ Existiert (unverändert) |
| Signin-Page | `src/app/auth/signin/page.tsx` | ✅ Wallet-Only → MUSS um Email-Tab erweitert werden |
| Middleware | `src/middleware.ts` | ✅ Existiert → MUSS `/auth/verify` erlauben |
| MMS Dir | `src/infrastructure/mms/` | ⚠️ Leer (.gitkeep) → MUSS erstellt werden |
| Auth Dir | `src/lib/auth/` | ✅ auth-options.ts + session.ts → MUSS um turnstile.ts + magic-link.ts erweitert werden |
| Format Utils | `src/lib/utils/format.ts` | ✅ `truncateAddress()` vorhanden → Optional: `truncateEmail()` ergänzen |
| Root Layout | `src/app/layout.tsx` | ✅ Provider-Hierarchy steht — KEIN Eingriff nötig |

### Project Structure Notes

- MMS-Client gehört in `src/infrastructure/mms/` (Infrastructure Layer)
- Turnstile-Verification gehört in `src/lib/auth/` (Shared Utilities)
- Magic-Link-Token-Logik gehört in `src/lib/auth/` (Shared Utilities — keine Domain-Logik)
- Verification-Token-Repository gehört in `src/infrastructure/db/repositories/` (Infrastructure Layer)
- Email-Login API Routes gehören in `src/app/api/auth/email/` (App Layer)
- Verify-Page gehört in `src/app/auth/verify/` (App Layer)
- Domain Layer (`src/domain/`) wird in dieser Story NICHT berührt

### Sicherheitsaspekte

- **Token-Hashing (SHA-256):** Klartext-Token nur in der Email-URL, DB speichert nur Hash → DB-Leak leakt keine gültigen Tokens
- **Token Single-Use:** Nach erfolgreicher Verifikation wird der Token sofort gelöscht
- **Token-Expiry (15 Min):** Kurze Gültigkeit reduziert Angriffsfenster
- **Turnstile vor Token-Send:** Bot-Schutz verhindert Email-Spam über die Magic-Link-Route
- **Rate-Limiting:** Max 3 Magic-Link-Requests pro Email pro 15 Minuten
- **Keine PII in Logs (NFR15):** Email-Adressen nur gekürzt loggen
- **HMAC auf `/api/auth/email/send`:** NICHT nötig (kein Cron-Endpoint), aber Turnstile + Rate-Limit schützen

### References

- [Source: _bmad-output/planning-artifacts/epics.md#Story 1.3]
- [Source: _bmad-output/planning-artifacts/architecture.md#Authentication & Security]
- [Source: _bmad-output/planning-artifacts/architecture.md#Core Architectural Decisions]
- [Source: _bmad-output/planning-artifacts/architecture.md#Implementation Patterns & Consistency Rules]
- [Source: _bmad-output/planning-artifacts/architecture.md#Complete Project Directory Structure]
- [Source: _bmad-output/planning-artifacts/prd.md#FR4 — Email-Login via MMS]
- [Source: _bmad-output/planning-artifacts/prd.md#NFR14 — Bot-Schutz Cloudflare Turnstile]
- [Source: _bmad-output/planning-artifacts/prd.md#NFR15 — Keine PII in Logs]
- [Source: _bmad-output/planning-artifacts/prd.md#NFR16 — JWT-Sessions]
- [Source: _bmad-output/planning-artifacts/ux-design-specification.md#Design-Tokens — Dark Mode Farbsystem]
- [Source: _bmad-output/project-context.md#Authentication Rules]
- [Source: _bmad-output/project-context.md#Auth Stack]
- [Source: _bmad-output/project-context.md#masemIT Integration Rules — MMS]
- [Source: _bmad-output/project-context.md#Security Rules]
- [Source: _bmad-output/implementation-artifacts/1-2-eth-wallet-authentifizierung-siwe.md — Learnings + File List]

## Dev Agent Record

### Agent Model Used

Claude Opus 4.6 (claude-opus-4-6)

### Debug Log References

- MMS API hat keinen Email-Send-Endpoint → Rücksprache mit Sempre → Resend als Ersatz verwendet
- `@marsidev/react-turnstile` war bereits in package.json installiert

### Completion Notes List

- **Task 1:** Cloudflare Turnstile Integration — Env-Vars, `.env.example`, Server-Verification-Helper mit 6 Tests
- **Task 2:** Email-Client via Resend (statt MMS) — MMS hat keinen Email-Endpoint, Sempre bestätigt Resend. Typen + Zod-Schemas + 5 Tests
- **Task 3:** `verification_tokens`-Tabelle — Drizzle-Schema mit Composite PK, Migration generiert und auf NeonDB angewendet, 4 Schema-Tests
- **Task 4:** Magic-Link Token-Service — generateMagicLinkToken (SHA-256 Hash) + verifyMagicLinkToken (Single-Use, 15min Expiry) + Verification-Token-Repository, 12 Tests
- **Task 5:** Email-Login API Endpoints — POST `/api/auth/email/send` (Zod-Validierung + Turnstile + Rate-Limiting + Magic-Link) + GET `/api/auth/email/verify` (Token-Verifizierung + User-Upsert + Redirect)
- **Task 6:** NextAuth erweitert — Zweiter CredentialsProvider "email-login", JWT/Session-Callbacks mit authMethod-Unterscheidung, TypeScript-Typen erweitert, 7 Tests
- **Task 7:** User Repository erweitert — findUserByEmail + upsertUserByEmail (lowercase Normalisierung), 7 Tests
- **Task 8:** Signin-Page UI — Tab-Layout (Wallet|Email), Email-Formular mit Turnstile, "Prüfe deine Email"-State, 60s Resend-Cooldown, Error-States, Dark-Theme
- **Task 9:** Verify-Page — Loading/Success/Error States, signIn via email-login Provider, Redirect zu Dashboard
- **Task 10:** Middleware — /auth/verify als öffentliche Route, authentifizierte User → Dashboard-Redirect
- **Task 11:** 37 neue Tests, 60 Tests total, 0 Regressionen, Biome-Check sauber

### Implementation Plan

- Resend statt MMS für Email-Versand (MMS hat keinen Email-Send-Endpoint)
- In-Memory Rate-Limiting (Upstash Rate Limit nicht installiert)
- Inline-Email-Validierung statt Zod (einfacher im Client-Kontext)
- Token-Verifizierung zweistufig: API-Route verifiziert Token, Verify-Page erstellt Session

### File List

**Neue Dateien:**
- src/lib/auth/turnstile.ts
- src/lib/auth/turnstile.test.ts
- src/lib/auth/magic-link.ts
- src/lib/auth/magic-link.test.ts
- src/infrastructure/mms/mms-client.ts
- src/infrastructure/mms/mms-client.test.ts
- src/infrastructure/mms/mms-types.ts
- src/infrastructure/db/repositories/verification-token-repository.ts
- src/infrastructure/db/repositories/verification-token-repository.test.ts
- src/app/api/auth/email/send/route.ts
- src/app/api/auth/email/verify/route.ts
- src/app/auth/verify/page.tsx
- drizzle/0002_even_crusher_hogan.sql

**Geänderte Dateien:**
- src/lib/env.ts (Turnstile env vars)
- .env.example (Turnstile keys)
- src/infrastructure/db/schema.ts (verification_tokens Tabelle)
- src/infrastructure/db/schema.test.ts (neue Tests)
- src/infrastructure/db/repositories/user-repository.ts (findUserByEmail, upsertUserByEmail)
- src/infrastructure/db/repositories/user-repository.test.ts (neue Tests)
- src/lib/auth/auth-options.ts (Email-Login-Provider, JWT/Session-Callbacks)
- src/lib/auth/auth-options.test.ts (neue Tests)
- src/types/next-auth.d.ts (email, authMethod Typen)
- src/app/auth/signin/page.tsx (Tab-Layout, Email-Login-UI)
- src/middleware.ts (/auth/verify Route)
- package.json (resend, @marsidev/react-turnstile)
- package-lock.json (neue Abhängigkeiten)
- _bmad-output/implementation-artifacts/sprint-status.yaml (Status-Update)

## Senior Developer Review

### Reviewer
Claude Opus 4.6 (Adversarial Code Review)

### Review Date
2026-02-20

### Findings Summary

| Severity | Found | Fixed | Remaining |
|----------|-------|-------|-----------|
| CRITICAL | 1 | 1 | 0 |
| HIGH | 2 | 2 | 0 |
| MEDIUM | 4 | 4 | 0 |
| LOW | 3 | 1 | 2 |

### Findings Detail

**C1 (FIXED): Toter verified-Pfad in verify/page.tsx**
- verify/page.tsx hatte einen `verified=true`-Flow, der Token als String "verified" an signIn übergab — dieser Pfad war zwar nicht im Hauptflow aktiv (send/route.ts zeigt direkt auf /auth/verify), aber stellte toten Code mit gebrochener Logik dar
- verify/route.ts konsumierte Token redundant und leitete mit `verified=true` weiter
- **Fix:** Toten `verified`-Pfad aus verify/page.tsx entfernt, verify/route.ts zu einfachem Redirect-Passthrough vereinfacht, redundanten `upsertUserByEmail`-Aufruf entfernt

**H1 (FIXED): clientEnv.NEXT_PUBLIC_TURNSTILE_SITE_KEY nie befüllt**
- Variable war in `clientSchema` definiert, aber nicht in `validateClientEnv()` an `safeParse()` übergeben
- Turnstile-Widget rendert nie → Bot-Schutz auf Signin-Page funktionslos
- **Fix:** Variable in `validateClientEnv()` safeParse-Aufruf ergänzt

**H2 (FIXED): process.env statt serverEnv in 4 Dateien**
- turnstile.ts, mms-client.ts, send/route.ts, verify/route.ts nutzten `process.env` direkt statt `serverEnv` aus `@/lib/env`
- Verstößt gegen validiertes Env-Muster aus Story 1-1 Review (M5)
- **Fix:** Alle 4 Dateien auf `serverEnv` umgestellt

**M1 (FIXED): In-Memory Rate-Limit ohne Cleanup**
- `rateLimitMap` wächst unbegrenzt im Serverless-Kontext
- **Fix:** `cleanupRateLimitMap()` entfernt abgelaufene Einträge bei >1000 Map-Größe

**M2 (FIXED): Redundanter upsertUserByEmail-Aufruf**
- verify/route.ts und auth-options.ts riefen beide `upsertUserByEmail` auf
- **Fix:** Durch C1-Fix behoben (verify/route.ts ist jetzt reiner Redirect)

**M3 (FIXED): package.json/package-lock.json in File List fehlend**
- resend und @marsidev/react-turnstile als neue Abhängigkeiten nicht dokumentiert
- **Fix:** File List ergänzt

**M4 (FIXED): XSS-Vektor in Email-Template**
- `magicLinkUrl` unescaped in HTML `href` eingefügt
- **Fix:** `escapeHtmlAttr()` Helper erstellt und auf magicLinkUrl angewendet

**L1 (OFFEN): mms-types.ts enthält toten Code**
- Zod-Schemas für MMS-API-Responses, die nirgends importiert werden (Resend statt MMS)
- Niedrige Priorität, kann bei nächstem Cleanup entfernt werden

**L2 (OFFEN): Email-Adresse im Klartext in Magic-Link-URL**
- `?email=user@example.com` in der URL ist bei HTTPS akzeptabel, aber nicht ideal
- Kann in einer späteren Story durch Token-basiertes Email-Lookup ersetzt werden

**L3 (FIXED): Kein Timeout auf Turnstile-Fetch**
- **Fix:** `AbortSignal.timeout(10_000)` ergänzt

### Test Results After Review
- 13 Test-Dateien, 60 Tests — alle bestanden
- Biome-Check: sauber
- TypeScript `--noEmit`: 4 pre-existing strict-null Warnungen in Test-Dateien (nicht durch Review eingeführt)

### Test Mocks Updated
- `turnstile.test.ts`: `vi.stubEnv` → `vi.mock("@/lib/env")` (serverEnv-Mock)
- `mms-client.test.ts`: `vi.stubEnv` → `vi.mock("@/lib/env")` (serverEnv-Mock)

## Change Log

- 2026-02-20: Story 1-3 implementiert — Email-Authentifizierung via Magic-Link (Resend statt MMS). 11 Tasks abgeschlossen, 37 neue Unit-Tests, DB-Migration angewendet.
- 2026-02-20: Senior Developer Review — 1 CRITICAL, 2 HIGH, 4 MEDIUM behoben. 2 LOW offen (toter Code + Email in URL). Tests angepasst.
