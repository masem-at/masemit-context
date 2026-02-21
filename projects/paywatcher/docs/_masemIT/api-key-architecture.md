# PayWatcher API Key Architektur

## Übersicht: Zwei Keys, zwei Zwecke

PayWatcher verwendet zwei verschiedene MMS API Keys mit unterschiedlichen Aufgaben:

```mermaid
flowchart LR
    subgraph Browser["Browser"]
        Login["Login-Seite"]
        Dashboard["Dashboard"]
    end

    subgraph Vercel["PayWatcher Frontend (Vercel)"]
        AuthCallback["Auth.js Callback"]
        RouteHandlers["API Route Handlers"]
        AdminKey[("MMS_API_KEY<br/><b>Admin Key</b><br/>(Env Var)")]
        TenantKeys[("tenant_keys<br/><b>Tenant Key</b><br/>(Neon DB, JWE)")]
    end

    subgraph MMS["MMS Backend (api.masem.at)"]
        direction TB
        AuthEndpoint["POST /v1/auth/login<br/>Scope: parties:read"]
        PaywatcherAPI["GET/PATCH /v1/paywatcher/*<br/>Scope: paywatcher:read, ..."]
        PaymentsAPI["GET/POST /v1/payments/*<br/>Scope: payments:read, ..."]
    end

    Login -->|"1. Email"| AuthCallback
    AuthCallback -.->|"liest"| AdminKey
    AuthCallback -->|"2. x-api-key: Admin Key"| AuthEndpoint
    AuthEndpoint -->|"3. Party + Tenants"| AuthCallback

    Dashboard -->|"4. API Request"| RouteHandlers
    RouteHandlers -.->|"5. entschlüsselt"| TenantKeys
    RouteHandlers -->|"6. x-api-key: Tenant Key"| PaywatcherAPI
    RouteHandlers -->|"6. x-api-key: Tenant Key"| PaymentsAPI

    linkStyle 2 stroke:#3b82f6,stroke-width:3px
    linkStyle 3 stroke:#3b82f6,stroke-width:3px
    linkStyle 5 stroke:#a855f7,stroke-width:3px
    linkStyle 6 stroke:#a855f7,stroke-width:3px
    linkStyle 7 stroke:#a855f7,stroke-width:3px

    style AdminKey fill:#1e40af,color:#fff
    style TenantKeys fill:#7c3aed,color:#fff
    style AuthEndpoint fill:#1e40af,color:#fff,stroke:#3b82f6,stroke-width:2px
    style PaywatcherAPI fill:#7c3aed,color:#fff,stroke:#a855f7,stroke-width:2px
    style PaymentsAPI fill:#7c3aed,color:#fff,stroke:#a855f7,stroke-width:2px
```

## Die zwei Keys im Detail

### 1. Admin Key (`MMS_API_KEY`) — blau im Diagramm

| Eigenschaft | Wert |
|---|---|
| **Gespeichert in** | `.env.local` (lokal) / Vercel Environment Variables (Produktion) |
| **Verwendet von** | Auth.js `signIn` + `jwt` Callback (serverseitig) |
| **Aufgabe** | Prüft beim Login ob eine Party mit PayWatcher-Tenant in MMS existiert |
| **MMS Endpoints** | `POST /v1/auth/login`, `/v1/admin/paywatcher/*` |
| **Benötigte Scopes** | `parties:read` (Login), `admin:read` + `admin:write` (Tenant-Verwaltung) |
| **Zuordnung in MMS** | Keiner Party/Tenant zugeordnet — ist ein Server-Level Key |

### 2. Tenant Key (`tenant_keys` DB) — lila im Diagramm

| Eigenschaft | Wert |
|---|---|
| **Gespeichert in** | Neon Postgres `tenant_keys` Tabelle, JWE-verschlüsselt mit `AUTH_SECRET` |
| **Verwendet von** | Alle API Route Handler im Dashboard (via `requireAuth()` → `getTenantApiKey()`) |
| **Aufgabe** | Authentifiziert API-Calls an MMS für den jeweiligen Tenant |
| **MMS Endpoints** | `/v1/paywatcher/*`, `/v1/payments/*`, `/v1/webhooks/*` |
| **Benötigte Scopes** | Zugriff auf alle PayWatcher-relevanten Endpoints |
| **Zuordnung in MMS** | Dem jeweiligen Tenant zugeordnet (z.B. `masemit`) |

## MMS Datenbank-Zusammenhänge

### Tabellen

```mermaid
erDiagram
    auth_api_keys {
        uuid id PK
        api_key_project_enum project "hoki_help | tellingcube | chainsights | masem_at | kurzum_app | paywatcher | admin"
        varchar key_hash "SHA-256 Hash des Plaintext Keys"
        varchar key_prefix "Erste 10 Zeichen (z.B. mms_paywat)"
        varchar name "Beschreibender Name"
        text[] scopes "Array von Berechtigungen"
        varchar tenant_slug "FK-artig auf pw_config.tenant_slug (nullable)"
        boolean is_active "true = aktiv, false = revoked"
        timestamp last_used_at
        timestamp created_at
    }

    pw_config {
        uuid id PK
        source_project_enum source_project
        varchar tenant_slug UK "Eindeutiger Tenant-Identifier"
        varchar tenant_name "Anzeigename"
        varchar contact_email
        varchar tier "free | pro | ..."
        text webhook_url
        text webhook_secret
        text deposit_address
        boolean enabled
        timestamp created_at
    }

    auth_api_keys }o--o| pw_config : "tenant_slug"
```

### Key-Typen in `auth_api_keys`

Es gibt drei logische Key-Typen, die sich durch `project`, `scopes` und `tenant_slug` unterscheiden:

| Key-Typ | `project` | `tenant_slug` | `scopes` | Verwendung |
|---|---|---|---|---|
| **Admin Key** | `paywatcher` | `NULL` | `parties:read`, `admin:read`, `admin:write` | PayWatcher Frontend: Login-Flow (`POST /v1/auth/login` braucht `parties:read`), Tenant-Verwaltung (`/v1/admin/paywatcher/*` braucht `admin:*`) |
| **Tenant Key** | `paywatcher` | z.B. `masemit` | `payments:read`, `payments:write` | PayWatcher Frontend: Dashboard-API-Calls pro Tenant |
| **Andere Projekt-Keys** | z.B. `chainsights` | `NULL` | Projektspezifisch | Andere MMS-Konsumenten (nicht PayWatcher-relevant) |

### Scope-Unterschied: `admin:*` vs. `payments:*`

| Scope | Sichtbarkeit | Endpunkte | Zweck |
|---|---|---|---|
| `admin:read` | **Cross-Tenant** (alle Tenants) | `GET /v1/admin/paywatcher/tenants`, `/payments`, `/health` | Plattform-Verwaltung |
| `admin:write` | **Cross-Tenant** | `POST/PATCH /v1/admin/paywatcher/tenants` | Tenants anlegen/updaten |
| `payments:read` | **Eigener Tenant** nur | `GET /v1/payments/*`, `GET /v1/paywatcher/config` | Tenant-Dashboard Lesezugriff |
| `payments:write` | **Eigener Tenant** nur | `POST /v1/payments/*`, `PATCH /v1/paywatcher/config` | Payments erstellen, Config updaten |

### Tenant-Zuordnung

Die Verknüpfung `auth_api_keys.tenant_slug` → `pw_config.tenant_slug` ist eine **logische Referenz** (kein DB-Level Foreign Key). Das MMS Auth-Middleware resolvet den Tenant anhand des `tenant_slug` im API Key:

1. Request kommt mit `x-api-key: mms_paywatcher_xxx`
2. Middleware hasht den Key, findet den Eintrag in `auth_api_keys`
3. Wenn `tenant_slug` gesetzt → Scoped auf diesen Tenant (Tenant Key)
4. Wenn `tenant_slug` NULL → Cross-Tenant Zugriff je nach Scopes (Admin Key)

## Einrichtung

### Admin Key setzen

In `.env.local` bzw. Vercel Environment Variables:

```
MMS_API_KEY=mms_admin_xxxxxxxx
```

### Tenant Key setzen

Via CLI-Script (verschlüsselt automatisch mit `AUTH_SECRET`):

```bash
npx tsx scripts/set-tenant-key.ts <tenant_slug> <api_key>

# Beispiel:
npx tsx scripts/set-tenant-key.ts masemit mms_payw_3a4290...
```

> **Wichtig:** Nie den rohen API Key direkt in die `tenant_keys`-Tabelle schreiben.
> Immer über das Script gehen, das den Key als JWE verschlüsselt.

## Request-Flow im Detail

```mermaid
sequenceDiagram
    participant U as User (Browser)
    participant PW as PayWatcher (Vercel)
    participant DB as Neon Postgres
    participant MMS as MMS Backend

    Note over U,MMS: Login-Flow (Admin Key)
    U->>PW: Email eingeben → signIn("resend", {email})
    PW->>MMS: POST /v1/auth/login<br/>x-api-key: MMS_API_KEY (Admin)
    MMS-->>PW: {party_id, tenants: [{tenant_slug: "masemit"}]}
    PW->>U: Magic Link per Email
    U->>PW: Magic Link klicken
    PW->>PW: JWT Session erstellen (tenantSlug: "masemit")
    PW->>U: Redirect → /dashboard

    Note over U,MMS: Dashboard-Flow (Tenant Key)
    U->>PW: Dashboard öffnet → GET /api/tenant
    PW->>PW: requireAuth() → Session lesen
    PW->>DB: SELECT encrypted_api_key<br/>WHERE tenant_slug = "masemit"
    DB-->>PW: JWE-verschlüsselter Key
    PW->>PW: JWE entschlüsseln → Klartext Tenant Key
    PW->>MMS: GET /v1/paywatcher/config<br/>x-api-key: Tenant Key
    MMS-->>PW: Tenant Config
    PW-->>U: Dashboard-Daten
```
