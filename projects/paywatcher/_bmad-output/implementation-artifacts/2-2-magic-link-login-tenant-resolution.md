# Story 2.2: Magic Link Login & Tenant-Resolution

Status: done

<!-- Note: Validation is optional. Run validate-create-story for quality check before dev-story. -->

## Story

As a Tenant,
I want mich per Magic Link (Email) im Dashboard einloggen koennen,
so that ich ohne API Key oder Passwort einfach per Email-Klick auf mein Dashboard zugreifen kann.

## Acceptance Criteria

1. **Given** ich bin auf der Login-Seite (/login) **When** die Seite laedt **Then** sehe ich ein Eingabefeld fuer meine Email-Adresse mit Placeholder "your@email.com" und einen "Send Magic Link" Button

2. **Given** ich gebe meine Email ein und klicke "Send Magic Link" **When** Auth.js den Email Provider triggert **Then** wird ein Magic Link via Resend an meine Email gesendet **And** ich sehe: "Check your email — we sent you a sign-in link."

3. **Given** ich klicke den Magic Link in meiner Email **When** Auth.js den Link verifiziert **Then** ruft der signIn Callback POST /v1/auth/login mit `{ email, module: "paywatcher" }` auf **And** die Response enthaelt: `{ party_id, display_name, tenants: [{ tenant_slug, role }] }`

4. **Given** der User genau einem Tenant zugeordnet ist (tenants.length === 1) **Then** wird der tenant_slug automatisch in die Session geschrieben **And** der API Key wird aus der Neon Postgres DB geladen (tenant_keys Tabelle, verschluesselt) **And** ich werde zum Dashboard redirected

5. **Given** der User mehreren Tenants zugeordnet ist (tenants.length > 1) **Then** sehe ich eine Tenant-Auswahl-Seite (/auth/resolve) mit den verfuegbaren Tenants **And** nach Auswahl wird der tenant_slug in die Session geschrieben und der API Key aus der DB geladen

6. **Given** die Email keinem Tenant zugeordnet ist (tenants leer oder 404) **Then** sehe ich: "No PayWatcher account found for this email. Request access first." **And** ein Link zum Request Access Formular

7. **Given** ich nach dem Login eine Dashboard-Route aufrufe **Then** enthaelt die Session tenant_slug, tenant_name, contact_email und den verschluesselten API Key fuer MMS-Calls

## Tasks / Subtasks

- [x] Task 1: Neon Postgres DB + Drizzle ORM Setup (AC: #4, #7)
  - [x] 1.1 Packages installieren: `drizzle-orm`, `@neondatabase/serverless`, `@auth/drizzle-adapter`, `drizzle-kit` (dev)
  - [x] 1.2 `src/lib/db/index.ts` erstellen — Drizzle Client mit `neon()` HTTP Driver (aus `@neondatabase/serverless`), Schema-Import
  - [x] 1.3 `src/lib/db/schema.ts` erstellen — Auth.js Tables (users, accounts, verificationTokens) + Custom `tenant_keys` Tabelle (tenant_slug PK, encrypted_api_key, created_at, updated_at)
  - [x] 1.4 `drizzle.config.ts` erstellen — dialect: "postgresql", schema: "./src/lib/db/schema.ts", dbCredentials: url aus DATABASE_URL
  - [x] 1.5 `.env.example` aktualisieren: `DATABASE_URL` (Neon Postgres Connection String) hinzufuegen
  - [ ] 1.6 DB-Schema via `npx drizzle-kit push` auf Neon deployen (oder Migration generieren) — Erfordert echte DATABASE_URL, manuell durch Mario

- [x] Task 2: Tenant API Key Management (AC: #4, #7)
  - [x] 2.1 `src/lib/api/tenant-keys.ts` erstellen — `getTenantApiKey(tenantSlug): Promise<string>` (DB Lookup + Decrypt via JOSE), `setTenantApiKey(tenantSlug, apiKey): Promise<void>` (Encrypt + Upsert)
  - [x] 2.2 Encryption beibehalten: bestehende `encryptApiKey()` / `decryptApiKey()` Funktionen aus `src/lib/auth.ts` in `tenant-keys.ts` verschieben oder als Shared Utility extrahieren (nutzt AUTH_SECRET als Encryption Key, JOSE EncryptJWT/jwtDecrypt, A256GCM)
  - [x] 2.3 `requireAuth()` Helper in `src/lib/auth.ts` aktualisieren: API Key aus DB holen statt aus JWT Token (via `getTenantApiKey(session.tenantSlug)`)

- [x] Task 3: Auth.js v5 von Credentials auf Magic Link migrieren (AC: #1, #2, #3, #4, #5, #6, #7)
  - [x] 3.1 Resend Email Provider konfigurieren: `import Resend from "next-auth/providers/resend"`, `from: process.env.RESEND_FROM_EMAIL`, optional: Custom `sendVerificationRequest` fuer branded Email-Template
  - [x] 3.2 DrizzleAdapter einbinden: `adapter: DrizzleAdapter(db, { usersTable, accountsTable, verificationTokensTable })`, Session Strategy explizit auf `"jwt"` setzen (weil Adapter sonst auf DB-Sessions defaulted)
  - [x] 3.3 Credentials Provider entfernen (id: "api-key" komplett raus)
  - [x] 3.4 `signIn` Callback implementieren: POST /v1/auth/login aufrufen mit `{ email: user.email, module: "paywatcher" }`, tenants[] auswerten (0 → false/redirect, 1 → auto-select, >1 → Tenant-Auswahl pending)
  - [x] 3.5 `jwt` Callback aktualisieren: partyId, displayName, tenantSlug, tenantName, contactEmail ins Token schreiben (KEIN API Key im JWT — Key kommt aus DB)
  - [x] 3.6 `session` Callback aktualisieren: partyId, displayName, tenantSlug, tenantName, contactEmail in Session exponieren
  - [x] 3.7 `pages` Config: signIn → "/login", evtl. verifyRequest → Custom "check your email" Page

- [x] Task 4: Login-Seite umbauen (AC: #1, #2)
  - [x] 4.1 `src/components/landing/auth-form.tsx` komplett umbauen: API-Key-Input entfernen, Email-Input mit Placeholder "your@email.com" + "Send Magic Link" Button
  - [x] 4.2 States: idle → submitting (Button disabled + Loading) → success ("Check your email — we sent you a sign-in link." + Hinweis "Didn't receive it? Check spam or [resend]" mit Resend-Button nach 30s)
  - [x] 4.3 Error State: "Could not send magic link. Please try again." (Inline unter Feld, rot, caption-size)
  - [x] 4.4 Link beibehalten: "No API key? Request access" → "Don't have an account? Request access"
  - [x] 4.5 signIn("resend", { email, redirectTo: "/dashboard" }) aufrufen statt signIn("api-key", ...)

- [x] Task 5: Tenant-Resolution & Multi-Tenant Support (AC: #3, #4, #5, #6)
  - [x] 5.1 Tenant-Resolution Logik: Im signIn Callback (oder als separater Step nach Magic Link Verification): POST /v1/auth/login Response auswerten
  - [x] 5.2 Single-Tenant (tenants.length === 1): tenant_slug automatisch in User-Objekt schreiben → jwt Callback uebernimmt in Token
  - [x] 5.3 Multi-Tenant (tenants.length > 1): User-Objekt mit `tenants[]` Array setzen, Redirect zu `/auth/resolve`
  - [x] 5.4 `src/app/(landing)/auth/resolve/page.tsx` erstellen: Tenant-Auswahl UI — Liste der verfuegbaren Tenants mit Name + Slug, Klick waehlt Tenant, schreibt tenant_slug via Server Action/API in JWT Session
  - [x] 5.5 No-Tenant (tenants.length === 0 oder 404): Redirect zu `/login?error=no-account` mit Fehlermeldung "No PayWatcher account found for this email." + Link zum Request Access Formular
  - [x] 5.6 API Key aus DB laden nach Tenant-Auswahl: `getTenantApiKey(tenantSlug)` aufrufen, Ergebnis NICHT in JWT speichern — Route Handlers holen Key on-demand aus DB

- [x] Task 6: Logout & Redirect (AC: #7)
  - [x] 6.1 Logout-Funktion sicherstellen: `signOut({ redirectTo: "/" })` leitet zur Landing Page
  - [x] 6.2 `/signup` Route weiterhin auf `/login` redirecten (kein eigener Signup-Flow)

- [x] Task 7: Types & Module Augmentation aktualisieren (AC: #7)
  - [x] 7.1 `src/types/auth.ts` aktualisieren: partyId, displayName, tenantSlug, tenantName, contactEmail (KEIN encryptedApiKey mehr — Key kommt aus DB)
  - [x] 7.2 `src/types/next-auth.d.ts` aktualisieren: Session.user, User, JWT Module Augmentation mit neuen Feldern
  - [x] 7.3 MMS Auth Types hinzufuegen: `MmsAuthLoginResponse` in `src/lib/api/types.ts` (party_id, display_name, tenants[])

- [x] Task 8: Build-Validierung und Qualitaetspruefung (Alle ACs)
  - [x] 8.1 `npm run build` muss fehlerfrei durchlaufen
  - [x] 8.2 `npm run type-check` muss fehlerfrei durchlaufen
  - [x] 8.3 `npm run lint` muss fehlerfrei durchlaufen (1 Warning: _request unused — akzeptabel)
  - [x] 8.4 Verifizieren: Landing Page bleibt `○ (Static)` im Build Output (SSG nicht gebrochen)
  - [x] 8.5 Verifizieren: Login-Seite zeigt Email-Input (nicht API Key)
  - [x] 8.6 Verifizieren: Dashboard-Routen sind weiterhin geschuetzt (Redirect zu /login ohne Session)
  - [ ] 8.7 Verifizieren: DB-Schema korrekt auf Neon deployed (users, accounts, verificationTokens, tenant_keys) — Erfordert echte DATABASE_URL
  - [x] 8.8 Verifizieren: `requireAuth()` holt API Key aus DB (nicht aus JWT)

## Dev Notes

### Kontext & Zweck

Dies ist die **zweite Story von Epic 2** (Authentication, Frontend DB & API Key Management). Story 2.1 hat die Dashboard Shell, Route Protection, MMS API Client und Auth.js v5 mit einem **Credentials Provider (API Key Login)** als Stub geliefert. Story 2.2 ersetzt diesen Stub komplett durch das finale Auth-System: **Magic Link via Resend + Parties API Tenant-Resolution + Neon Postgres DB**.

**Was PayWatcher ist:** Developer-First Verification API fuer Stablecoin-Zahlungen (USDC auf Base Network). Non-custodial, reiner Verification Layer. Das Frontend kommuniziert ueber Server-Side Route Handlers mit dem MMS Backend (api.masem.at). Die einzige eigene Datenbank (Neon Postgres) speichert Auth.js Verification Tokens und verschluesselte Tenant API Keys.

**Was diese Story liefert:**
- Neon Postgres DB Setup (Drizzle ORM, Serverless HTTP Driver)
- Auth.js v5 Migration von Credentials → Resend Magic Link Provider
- DrizzleAdapter fuer Verification Token Storage (Magic Link benoetigt DB!)
- MMS Parties API Integration (POST /v1/auth/login → Tenant-Resolution)
- tenant_keys Tabelle (tenant_slug → encrypted API Key in DB statt JWT)
- Multi-Tenant Support (Tenant-Auswahl bei >1 Tenant)
- Login-Seite Umbau (Email statt API Key)
- Logout → Landing Page Redirect

**Was diese Story NICHT beinhaltet:**
- Welcome-Screen & Onboarding (Story 2.3)
- API Key Anzeige & Scopes UI (Story 2.4 — bereits implementiert, unabhaengig)
- Admin-Zugang / Admin API Key (Phase 1b)
- Self-Service Signup / Account-Erstellung (Phase 2)
- Tenant-Switch im Dashboard-Header (optional future)

**Abhaengigkeiten:**
- Story 2.1 komplett abgeschlossen (Dashboard Shell, Auth.js v5 Stub, mmsApi(), Route Protection) ✅
- Neon Postgres Projekt muss erstellt sein (Mario richtet ein, DATABASE_URL als ENV)
- Resend API Key muss konfiguriert sein (RESEND_API_KEY bereits in .env.example)
- Mindestens ein Tenant + Party muss in MMS existieren (Mario onboardet manuell)

### KRITISCH: Architektur-Aenderung gegenueber Story 2.1

**Story 2.1 implementierte:** Credentials Provider mit API Key Login. Der API Key wurde direkt als JWE verschluesselt im JWT Token gespeichert. `requireAuth()` entschluesselt den Key aus dem Token.

**Story 2.2 aendert fundamental:**
1. **Auth Provider:** Credentials (API Key) → Resend (Magic Link Email)
2. **Token Storage:** Kein API Key mehr im JWT — Key liegt verschluesselt in der DB (`tenant_keys` Tabelle)
3. **DB-Adapter:** Kein Adapter → DrizzleAdapter (fuer verificationTokens, users, accounts)
4. **Session-Inhalt:** JWT enthaelt jetzt partyId, displayName, tenantSlug, tenantName, contactEmail (KEIN encryptedApiKey)
5. **API Key Lookup:** `requireAuth()` holt Key aus DB via `getTenantApiKey(tenantSlug)` statt aus JWT

**ACHTUNG — Frueherer Versuch:** Die Git-History zeigt Commits `fc10c16` und `9d062a0` — ein frueherer Magic-Link-Versuch der zurueckgerollt wurde. Der aktuelle main-Branch hat den API-Key-Login. Diese Story implementiert Magic Link NEU und korrekt.

### Technische Anforderungen

**Neu zu installierende Packages:**
- `drizzle-orm` — Lightweight Type-Safe ORM fuer Neon Postgres
- `@neondatabase/serverless` — Neon HTTP Driver (kein Connection Pooling noetig auf Vercel)
- `@auth/drizzle-adapter` — Auth.js Drizzle Adapter (users, accounts, verificationTokens)
- `drizzle-kit` (devDependency) — Schema Management, Migrations, Push

**Bereits installierte und verwendbare Packages:**
- `next-auth@^5.0.0-beta.30` (Auth.js v5) — bleibt, nur Provider-Wechsel
- `resend@^6.9.2` — bereits installiert fuer Request Access Emails, jetzt auch fuer Magic Link
- `jose` — bereits da (via next-auth), wird fuer API Key Encryption weiterverwendet
- `@tanstack/react-query@^5.90.21` — unveraendert
- `zod@^4.3.6` — fuer Email-Validierung im Login-Form

**NICHT in dieser Story installieren:**
- `react-hook-form` — Login hat nur ein Email-Feld, Zod reicht
- `recharts` — Dashboard-Charts kommen in Phase 1b
- `@auth/prisma-adapter` — wir nutzen Drizzle, nicht Prisma

### KRITISCH: Auth.js v5 mit Magic Link + JWT Strategy

**Fundamentale Erkenntnis aus Web Research:**
Magic Link (Email Provider) benoetigt ZWINGEND einen DB-Adapter fuer Verification Token Storage. ABER: Die Session-Strategy kann trotzdem JWT sein! Man muss explizit `session: { strategy: "jwt" }` setzen, weil Auth.js bei vorhandenem Adapter auf `"database"` defaulted.

```typescript
// src/lib/auth.ts — KORREKTE Konfiguration
import NextAuth from "next-auth"
import Resend from "next-auth/providers/resend"
import { DrizzleAdapter } from "@auth/drizzle-adapter"
import { db } from "@/lib/db"
import { users, accounts, verificationTokens } from "@/lib/db/schema"

export const { auth, handlers, signIn, signOut } = NextAuth({
  adapter: DrizzleAdapter(db, {
    usersTable: users,
    accountsTable: accounts,
    verificationTokensTable: verificationTokens,
  }),
  session: { strategy: "jwt" }, // EXPLIZIT! Sonst defaulted Adapter auf DB-Sessions
  providers: [
    Resend({
      from: process.env.RESEND_FROM_EMAIL,
      // AUTH_RESEND_KEY wird automatisch aus ENV gelesen
    }),
  ],
  callbacks: {
    async signIn({ user }) {
      // POST /v1/auth/login fuer Tenant-Resolution
      // Siehe Task 3.4 fuer Details
      return true
    },
    jwt({ token, user }) {
      if (user) {
        token.partyId = user.partyId
        token.displayName = user.displayName
        token.tenantSlug = user.tenantSlug
        token.tenantName = user.tenantName
        token.contactEmail = user.email
      }
      return token
    },
    session({ session, token }) {
      session.user.partyId = token.partyId as string
      session.user.displayName = token.displayName as string
      session.user.tenantSlug = token.tenantSlug as string
      session.user.tenantName = token.tenantName as string
      session.user.contactEmail = token.contactEmail as string
      return session
    },
  },
  pages: {
    signIn: "/login",
  },
})
```

**Environment Variable fuer Resend:**
- `AUTH_RESEND_KEY` — Auth.js v5 liest diesen automatisch fuer den Resend Provider
- `RESEND_API_KEY` — existiert bereits fuer Request Access Emails (separater Use Case)
- **WICHTIG:** Beide Keys koennen identisch sein, aber Auth.js v5 erwartet `AUTH_RESEND_KEY` als ENV-Name

### Neon Postgres DB Schema

```typescript
// src/lib/db/schema.ts
import { pgTable, text, timestamp, integer, primaryKey } from "drizzle-orm/pg-core"
import type { AdapterAccountType } from "next-auth/adapters"

// Auth.js Required Tables
export const users = pgTable("user", {
  id: text("id").primaryKey().$defaultFn(() => crypto.randomUUID()),
  name: text("name"),
  email: text("email").unique(),
  emailVerified: timestamp("emailVerified", { mode: "date" }),
  image: text("image"),
})

export const accounts = pgTable("account", {
  userId: text("userId").notNull().references(() => users.id, { onDelete: "cascade" }),
  type: text("type").$type<AdapterAccountType>().notNull(),
  provider: text("provider").notNull(),
  providerAccountId: text("providerAccountId").notNull(),
  refresh_token: text("refresh_token"),
  access_token: text("access_token"),
  expires_at: integer("expires_at"),
  token_type: text("token_type"),
  scope: text("scope"),
  id_token: text("id_token"),
  session_state: text("session_state"),
}, (account) => [
  primaryKey({ columns: [account.provider, account.providerAccountId] }),
])

export const verificationTokens = pgTable("verificationToken", {
  identifier: text("identifier").notNull(),
  token: text("token").notNull(),
  expires: timestamp("expires", { mode: "date" }).notNull(),
}, (vt) => [
  primaryKey({ columns: [vt.identifier, vt.token] }),
])

// Custom PayWatcher Table
export const tenantKeys = pgTable("tenant_keys", {
  tenantSlug: text("tenant_slug").primaryKey(),
  encryptedApiKey: text("encrypted_api_key").notNull(),
  createdAt: timestamp("created_at", { mode: "date" }).defaultNow().notNull(),
  updatedAt: timestamp("updated_at", { mode: "date" }).defaultNow().notNull(),
})
```

### Drizzle Client (Neon HTTP Driver)

```typescript
// src/lib/db/index.ts
import { neon } from "@neondatabase/serverless"
import { drizzle } from "drizzle-orm/neon-http"
import * as schema from "./schema"

const sql = neon(process.env.DATABASE_URL!)
export const db = drizzle({ client: sql, schema })
```

**WICHTIG:** `neon-http` Driver verwenden (nicht `neon-serverless` WebSocket). HTTP ist besser fuer Vercel Serverless Functions (schnellere Cold Starts, single queries). WebSocket nur bei interaktiven Transaktionen noetig.

### MMS Parties API — Tenant-Resolution Flow

**Endpoint:** POST /v1/auth/login
**Request:** `{ "email": "user@acme.com", "module": "paywatcher" }`
**Response:** `{ "party_id": "uuid", "display_name": "Acme Corp", "tenants": [{ "tenant_slug": "acme", "role": "admin" }] }`

**Implementierung im signIn Callback:**

```typescript
async signIn({ user }) {
  if (!user.email) return false

  try {
    const response = await fetch(`${process.env.MMS_API_URL}/v1/auth/login`, {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
        "x-api-key": process.env.MMS_API_KEY!, // Admin Key fuer Auth-Endpoint
      },
      body: JSON.stringify({ email: user.email, module: "paywatcher" }),
    })

    if (!response.ok) {
      // 404 = Email nicht bekannt → kein Account
      if (response.status === 404) return "/login?error=no-account"
      return false
    }

    const data = await response.json()
    const { party_id, display_name, tenants } = data.data

    if (!tenants || tenants.length === 0) {
      return "/login?error=no-account"
    }

    // Attach to user object for jwt callback
    user.partyId = party_id
    user.displayName = display_name

    if (tenants.length === 1) {
      user.tenantSlug = tenants[0].tenant_slug
      user.tenantName = tenants[0].tenant_name || display_name
      user.contactEmail = user.email
    } else {
      // Multi-tenant: redirect to resolution page
      // Store tenants temporarily for resolution
      user.tenants = tenants
      user.tenantSlug = "" // Will be set after resolution
    }

    return true
  } catch {
    return false
  }
}
```

**ACHTUNG: `mmsApi()` NICHT im signIn Callback verwenden!**
Der signIn Callback laeuft im Auth.js Kontext wo `mmsApi()` evtl. nicht korrekt initialisiert ist (kein Tenant API Key noetig, Admin Key wird benoetigt). Direkter `fetch` mit `MMS_API_KEY` (Admin Key) ist hier korrekt.

### Tenant-Auswahl (Multi-Tenant)

Wenn `tenants.length > 1`, muss der User einen Tenant waehlen. Dies ist ein Edge Case fuer den MVP (Mario onboardet manuell, die meisten User haben 1 Tenant).

**Ansatz:** Nach erfolgreichem Magic Link Login mit Multi-Tenant-Ergebnis:
1. JWT Token enthaelt `tenants[]` Array aber leeren `tenantSlug`
2. Dashboard Layout erkennt leeren `tenantSlug` und redirected zu `/auth/resolve`
3. `/auth/resolve` zeigt Tenant-Liste
4. User waehlt Tenant → Server Action aktualisiert JWT via `signIn()` mit neuem Token

### Login-Seite UX (aus UX-Spec)

**Form:** Ein einziges Email-Feld. Kein Name, kein Passwort, kein Company.
**Scope:** "Zwei Forms im gesamten Produkt — Email-Signup und Webhook-URL. Beide haben ein einziges Textfeld."
**Validation:** Inline unter dem Feld (rot, caption-size). Nicht als Toast.
**Error:** Feld-Border wird rot, Fehlertext darunter, verschwindet bei erneutem Tippen.
**Success:** "Check your email — we sent you a sign-in link." + Hinweis auf Spam + Resend-Button nach 30s.
**Magic Link expired:** "Link expired. Enter your email again."

### Project Structure Notes

**Neue Dateien:**
```
src/
├── lib/
│   ├── db/
│   │   ├── index.ts                # NEU: Drizzle Client (Neon HTTP)
│   │   └── schema.ts               # NEU: DB Schema (Auth.js + tenant_keys)
│   └── api/
│       └── tenant-keys.ts          # NEU: getTenantApiKey/setTenantApiKey
├── app/
│   └── (landing)/
│       └── auth/
│           └── resolve/
│               └── page.tsx         # NEU: Tenant-Auswahl (Multi-Tenant)
drizzle.config.ts                    # NEU: Drizzle Kit Konfiguration
```

**Geaenderte Dateien:**
```
src/
├── lib/
│   └── auth.ts                      # AENDERN: Credentials → Resend + DrizzleAdapter
├── lib/
│   └── api/
│       └── types.ts                 # AENDERN: MmsAuthLoginResponse hinzufuegen
├── types/
│   ├── auth.ts                      # AENDERN: partyId, tenantSlug, etc. (kein encryptedApiKey)
│   └── next-auth.d.ts               # AENDERN: Module Augmentation aktualisieren
├── components/
│   └── landing/
│       └── auth-form.tsx            # AENDERN: API Key → Email Input
├── app/
│   └── (dashboard)/
│       └── layout.tsx               # AENDERN: tenantSlug Check, evtl. Redirect zu /auth/resolve
.env.example                         # AENDERN: DATABASE_URL, AUTH_RESEND_KEY hinzufuegen
package.json                         # AENDERN: neue Dependencies
```

**Entfernte Dateien / Code:**
- Credentials Provider Block in `auth.ts` (id: "api-key")
- `encryptApiKey()` / `decryptApiKey()` aus `auth.ts` → verschoben nach `tenant-keys.ts`
- `encryptedApiKey` aus JWT Token und Session Types

**Alignment mit Architecture Doc:**
- `lib/db/index.ts` und `lib/db/schema.ts` → exakt wie im Architecture Doc definiert ✅
- `lib/api/tenant-keys.ts` → wie im Architecture Doc spezifiziert ✅
- DrizzleAdapter → wie im Architecture Doc unter "Auth & Data Architecture" ✅
- JWT Strategy mit DB Adapter → korrekt, Architecture Doc sagt "JWT Strategy" ✅

**Konflikte/Abweichungen:**
- Architecture Doc referenziert `middleware.ts` an einer Stelle — wir nutzen `proxy.ts` (Next.js 16 Pattern, bereits in Story 2.1 korrekt umgesetzt)
- Architecture Doc sagt "sessions Table" optional bei JWT Strategy — korrekt, wir lassen sessions Table weg

### References

- [Source: _bmad-output/planning-artifacts/epics.md#Story 2.2] — User Story und Acceptance Criteria
- [Source: _bmad-output/planning-artifacts/architecture.md#Auth & Data Architecture] — Magic Link + Parties + Neon DB Flow
- [Source: _bmad-output/planning-artifacts/architecture.md#Auth Pattern] — Auth.js v5, JWT Strategy, signIn Callback
- [Source: _bmad-output/planning-artifacts/architecture.md#API Client Pattern] — mmsApi(), Route Handlers
- [Source: _bmad-output/planning-artifacts/architecture.md#Structure Patterns] — lib/db/, lib/api/tenant-keys.ts
- [Source: _bmad-output/planning-artifacts/ux-design-specification.md#Flow 1: Signup-to-First-Webhook] — Magic Link UX Flow
- [Source: _bmad-output/planning-artifacts/ux-design-specification.md#Form Design Patterns] — Single-Field Form, Inline Validation
- [Source: _bmad-output/planning-artifacts/ux-design-specification.md#Error States for Auth] — Magic Link expired, Email nicht angekommen
- [Source: _bmad-output/planning-artifacts/ux-design-specification.md#Toast Patterns] — Netzwerk-Fehler als Toast, sonst Inline
- [Source: _bmad-output/planning-artifacts/prd.md#FR9, FR10, FR37] — Login, Logout, Magic Link Dashboard Login
- [Source: _bmad-output/implementation-artifacts/2-1-dashboard-shell-route-protection.md] — Previous Story Intelligence
- [Source: Auth.js Docs — Resend Provider] — https://authjs.dev/getting-started/providers/resend
- [Source: Auth.js Docs — Drizzle Adapter] — https://authjs.dev/getting-started/adapters/drizzle
- [Source: Drizzle ORM Docs — Neon Connection] — https://orm.drizzle.team/docs/connect-neon

## Dev Agent Record

### Agent Model Used

Claude Opus 4.6 (claude-opus-4-6)

### Debug Log References

- Build-Fehler 1: `neon()` warf Fehler bei fehlendem DATABASE_URL waehrend Build-Zeit → Loesung: Lazy DB-Initialisierung mit `getDb()` Funktion und conditional `db` Export
- Build-Fehler 2: DrizzleAdapter validiert DB-Typ eagerly → Loesung: Conditional Adapter (`db ? DrizzleAdapter(db, ...) : undefined`)
- Build-Fehler 3: `/auth/resolve` Seite mit `useSession()` schlug bei SSG fehl → Loesung: Umgebaut zu Server Component + Client Component Split (page.tsx + tenant-selector.tsx)
- Lint-Fehler: `setState` in useEffect → Loesung: State-Initialisierung direkt aus `searchParams` statt via Effect
- Type-Fehler: Auth.js v5 `encode()` erfordert `salt` Parameter → Cookie-Name als Salt uebergeben

### Completion Notes List

- Komplette Migration von Credentials (API Key) Login zu Magic Link (Resend Email Provider)
- Neon Postgres DB Setup mit Drizzle ORM (lazy initialization fuer Build-Kompatibilitaet)
- DB Schema: Auth.js Tables (user, account, verificationToken) + Custom tenant_keys Table
- Encryption-Logik von auth.ts nach tenant-keys.ts verschoben (encryptApiKey, decryptApiKey)
- requireAuth() holt API Key jetzt aus DB statt JWT Token
- Login-Seite komplett umgebaut: Email-Input, Magic Link Flow, Inline-Fehler, Resend-Button nach 30s
- Tenant-Resolution: signIn Callback ruft POST /v1/auth/login auf, Single-Tenant auto-select, Multi-Tenant Redirect zu /auth/resolve
- Tenant-Auswahl Seite (/auth/resolve): Server Component + Client Component, JWT Update via /api/auth/resolve-tenant
- Dashboard Layout erkennt fehlenden tenantSlug und redirected zu /auth/resolve bei Multi-Tenant
- JWT enthaelt kein encryptedApiKey mehr — nur partyId, displayName, tenantSlug, tenantName, contactEmail
- Zwei Subtasks offen: 1.6 (drizzle-kit push) und 8.7 (DB-Schema Verify) — erfordern echte DATABASE_URL von Mario

### Change Log

- 2026-02-18: Story 2.2 implementiert — Magic Link Login, Neon DB, Tenant-Resolution, Multi-Tenant Support
- 2026-02-18: Code Review (AI) — 4 HIGH, 5 MEDIUM, 2 LOW Issues gefunden. 9 Issues gefixt:
  - H1: .env.example Kommentare korrigiert (JWT→DB Storage, MMS_API_KEY Usage)
  - H2: encryptionSecret lazy init (Build-Safety, konsistent mit DB Pattern)
  - H3: Session Callback falsy guards → !== undefined (empty string "" fix)
  - H4: console.warn wenn DrizzleAdapter disabled (Silent Failure Prevention)
  - M1: tenant-selector.tsx nach components/landing/ verschoben (Architecture Compliance)
  - M2: Error-Feedback in TenantSelector bei API-Fehler hinzugefuegt
  - M3: Zod-Validierung fuer resolve-tenant/route.ts Body hinzugefuegt
  - M4: console.error Logging im signIn Callback catch Block
  - M5: Zod Email-Validierung in auth-form.tsx hinzugefuegt

### File List

Neue Dateien:
- src/lib/db/index.ts — Drizzle Client (Neon HTTP, lazy init)
- src/lib/db/schema.ts — DB Schema (Auth.js Tables + tenant_keys)
- src/lib/api/tenant-keys.ts — getTenantApiKey/setTenantApiKey (DB Lookup + JOSE Encryption)
- src/app/(landing)/auth/resolve/page.tsx — Tenant-Auswahl Server Component
- src/components/landing/tenant-selector.tsx — Tenant-Auswahl Client Component (verschoben aus app/ nach Review)
- src/app/api/auth/resolve-tenant/route.ts — POST Endpoint fuer Tenant-Auswahl (JWT Update)
- drizzle.config.ts — Drizzle Kit Konfiguration

Geaenderte Dateien:
- src/lib/auth.ts — Credentials → Resend Provider, DrizzleAdapter, signIn Callback (MMS Parties API), requireAuth() aus DB
- src/types/auth.ts — partyId, displayName hinzugefuegt, encryptedApiKey entfernt
- src/types/next-auth.d.ts — Module Augmentation aktualisiert (partyId, displayName, tenants[])
- src/lib/api/types.ts — MmsAuthLoginResponse hinzugefuegt
- src/components/landing/auth-form.tsx — Komplett umgebaut: Email-Input, Magic Link, Success/Error States
- src/app/(landing)/login/page.tsx — Metadata aktualisiert
- src/app/(dashboard)/layout.tsx — Multi-Tenant Check, Redirect zu /auth/resolve
- .env.example — DATABASE_URL, AUTH_RESEND_KEY hinzugefuegt
- package.json — drizzle-orm, @neondatabase/serverless, @auth/drizzle-adapter, drizzle-kit
