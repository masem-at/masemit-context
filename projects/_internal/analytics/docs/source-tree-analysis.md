# Source Tree Analysis - masemIT Analytics

> Generated: 2026-01-28 | Scan Level: Exhaustive

## Directory Structure

```
masemit-analytics/
│
├── 📄 Configuration Files
│   ├── package.json           # Dependencies & scripts (Next.js 16.1.4, React 19)
│   ├── tsconfig.json          # TypeScript config
│   ├── next.config.ts         # Next.js config (security headers, CORS)
│   ├── tailwind.config.ts     # Tailwind CSS + masemIT brand colors
│   ├── postcss.config.mjs     # PostCSS/Autoprefixer
│   ├── .env                   # Environment variables (gitignored)
│   ├── .env.example           # Environment template
│   └── .gitignore             # Git ignore rules
│
├── 📁 app/                    # Next.js App Router (ENTRY POINT)
│   ├── layout.tsx             # Root layout (dark theme, Inter font, noindex)
│   ├── page.tsx               # Homepage → redirects to /dashboard
│   ├── globals.css            # Global Tailwind styles
│   │
│   ├── 📁 api/                # API Routes
│   │   ├── 📁 events/
│   │   │   └── route.ts       # ⚡ tracker.js event ingestion (POST/GET)
│   │   ├── 📁 ingest/
│   │   │   └── route.ts       # 🔧 Vercel Drain webhook (legacy)
│   │   └── 📁 projects/
│   │       └── 📁 [id]/
│   │           └── route.ts   # 📋 Project management (GET/PATCH)
│   │
│   ├── 📁 components/         # React Components
│   │   ├── Favicon.tsx        # Favicon component
│   │   ├── FilterBar.tsx      # Date range filter (1/7/30/90 days + custom)
│   │   ├── ProjectSettingsForm.tsx  # Project settings editor
│   │   ├── RefreshButton.tsx  # Dashboard refresh button
│   │   └── WorldMap.tsx       # 🗺️ Geographic visualization (react-simple-maps)
│   │
│   └── 📁 dashboard/          # Dashboard Pages
│       ├── page.tsx           # 📊 Main dashboard (all projects)
│       ├── 📁 settings/
│       │   └── page.tsx       # ⚙️ Project settings page
│       └── 📁 [project]/
│           └── page.tsx       # 📈 Per-project detail view
│
├── 📁 lib/                    # Shared Libraries
│   └── schema.sql             # 🗃️ PostgreSQL database schema
│
├── 📁 public/                 # Static Assets
│   └── tracker.js             # 📡 Self-hosted analytics tracker
│
├── 📁 types/                  # TypeScript Type Definitions
│   └── react-simple-maps.d.ts # Types for world map library
│
├── 📁 docs/                   # Generated Documentation
│   ├── index.md               # Master index
│   ├── project-context.md     # AI Agent quick reference
│   ├── architecture.md        # System design
│   ├── api-contracts.md       # API specifications
│   ├── data-models.md         # Database schema
│   ├── development-guide.md   # Setup & workflows
│   ├── source-tree-analysis.md # This file
│   └── project-scan-report.json # Workflow state
│
├── 📁 _bmad/                  # BMAD Framework (development tooling)
│
└── 📁 node_modules/           # Dependencies (gitignored)
```

## Critical Paths

| Path | Purpose | Key Files |
|------|---------|-----------|
| `app/` | Application entry point | `layout.tsx`, `page.tsx` |
| `app/api/events/` | tracker.js ingestion | `route.ts` |
| `app/api/ingest/` | Vercel Drain (legacy) | `route.ts` |
| `app/api/projects/` | Project management | `[id]/route.ts` |
| `app/components/` | UI Components | `FilterBar.tsx`, `WorldMap.tsx` |
| `app/dashboard/` | Dashboard UI | `page.tsx`, `[project]/page.tsx` |
| `public/` | Static assets | `tracker.js` |
| `lib/` | Shared code & schema | `schema.sql` |

## Entry Points

| Entry Point | File | Description |
|-------------|------|-------------|
| **Web App** | `app/layout.tsx` | Root layout, wraps all pages |
| **Homepage** | `app/page.tsx` | Redirects to `/dashboard` |
| **API (tracker.js)** | `app/api/events/route.ts` | Primary event ingestion |
| **API (legacy)** | `app/api/ingest/route.ts` | Vercel Drain webhook |
| **Database** | `lib/schema.sql` | Schema initialization script |
| **Tracker** | `public/tracker.js` | Client-side analytics script |

## Data Flow

```
┌─────────────────┐     ┌──────────────────┐     ┌─────────────┐
│   tracker.js    │────▶│   /api/events    │────▶│   NeonDB    │
│  (4 projects)   │     │   (CORS check)   │     │  (events)   │
└─────────────────┘     └──────────────────┘     └──────┬──────┘
                                                        │
┌─────────────────┐     ┌──────────────────┐            │
│  Vercel Drains  │────▶│   /api/ingest    │────────────┤
│   (legacy)      │     │  (HMAC verify)   │            │
└─────────────────┘     └──────────────────┘            │
                                                        │
                                                        ▼
┌─────────────────┐     ┌──────────────────┐     ┌─────────────┐
│    Browser      │◀────│    /dashboard    │◀────│   SQL Query │
│   (User View)   │     │  (Server Comp)   │     │  (via neon) │
└─────────────────┘     └──────────────────┘     └─────────────┘
```

## File Count by Type

| Extension | Count | Purpose |
|-----------|-------|---------|
| `.tsx` | 9 | React/Next.js pages & components |
| `.ts` | 5 | Config, API routes, types |
| `.css` | 1 | Global styles |
| `.sql` | 1 | Database schema |
| `.js` | 1 | tracker.js (client-side) |
| `.json` | 2 | Package & TS config |
| `.mjs` | 1 | PostCSS config |

**Total Source Files:** ~20 (excluding node_modules, .git)

## Key Components

### tracker.js (public/tracker.js)

Self-hosted analytics tracker with:
- Owner exclusion (`?owner=sempre` or `/admin` path)
- UUID v4 session/device ID generation
- Device/browser/OS detection
- SPA navigation hooks (pushState, replaceState, popstate, hashchange)
- sendBeacon with text/plain (CORS-safelisted)
- `window.masemit.track(eventName, props)` API

### WorldMap (app/components/WorldMap.tsx)

Geographic visualization using:
- react-simple-maps (ComposableMap, Geographies, Geography)
- d3-scale for color interpolation
- ISO 3166-1 alpha-2 to numeric code mapping
- Hover tooltips with country flags

### FilterBar (app/components/FilterBar.tsx)

Date range selection:
- Preset: Today, 7 days, 30 days, 90 days
- Custom: 1-365 days input
- URL-based state (`?range=30`)
- Uses React `useTransition` for pending states

## Notes

- **Components folder exists** - UI components extracted (v0.2.0)
- **No tests** - Test infrastructure not yet implemented
- **No CI/CD** - Deploys via Vercel Git integration
- **Vienna timezone** - Dashboard uses Europe/Vienna for date filtering
- **Dual data sources** - tracker.js (primary) + Vercel Drains (legacy)
