# Development Guide - masemIT Analytics

> Generated: 2026-01-28

## Prerequisites

| Requirement | Version | Notes |
|-------------|---------|-------|
| Node.js | 22.x+ | LTS recommended |
| npm | 10.x+ | Comes with Node.js |
| PostgreSQL | - | Via NeonDB (cloud) |

## Quick Start

```bash
# 1. Clone repository
git clone <repo-url>
cd masemit-analytics

# 2. Install dependencies
npm install

# 3. Configure environment
cp .env.example .env
# Edit .env with your credentials

# 4. Initialize database (requires DATABASE_URL)
npm run db:push

# 5. Start development server
npm run dev
```

Open [http://localhost:3000](http://localhost:3000)

## Environment Variables

| Variable | Required | Description |
|----------|----------|-------------|
| `DATABASE_URL` | Yes | NeonDB connection string |
| `DRAIN_SECRET` | Yes* | Vercel Drain signature secret |
| `PROJECT_MASEMIT` | No | Vercel Project ID for masem.at |
| `PROJECT_CHAINSIGHTS` | No | Vercel Project ID for ChainSights |
| `PROJECT_TELLINGCUBE` | No | Vercel Project ID for tellingCube |
| `PROJECT_HOKIHELP` | No | Vercel Project ID for hoki.help |

*Required in production for security

### Example `.env`

```env
DATABASE_URL="postgresql://user:pass@ep-xxx.eu-central-1.aws.neon.tech/neondb?sslmode=require"
DRAIN_SECRET="your-vercel-drain-secret"
PROJECT_MASEMIT="prj_xxxx"
PROJECT_CHAINSIGHTS="prj_xxxx"
PROJECT_TELLINGCUBE="prj_xxxx"
PROJECT_HOKIHELP="prj_xxxx"
```

## NPM Scripts

| Command | Description |
|---------|-------------|
| `npm run dev` | Start development server (with hot reload) |
| `npm run build` | Build for production |
| `npm run start` | Start production server |
| `npm run lint` | Run ESLint |
| `npm run db:push` | Initialize/update database schema |

## Project Structure

```
app/
├── layout.tsx          # Root layout (edit for global changes)
├── page.tsx            # Homepage (redirects to /dashboard)
├── globals.css         # Tailwind imports & custom styles
├── api/
│   ├── events/
│   │   └── route.ts    # tracker.js event ingestion (primary)
│   ├── ingest/
│   │   └── route.ts    # Vercel Drain (legacy)
│   └── projects/
│       └── [id]/
│           └── route.ts # Project management
├── components/
│   ├── FilterBar.tsx   # Date range filter
│   ├── WorldMap.tsx    # Geographic visualization
│   ├── ProjectSettingsForm.tsx
│   └── RefreshButton.tsx
└── dashboard/
    ├── page.tsx        # Main dashboard
    ├── settings/
    │   └── page.tsx    # Project settings
    └── [project]/
        └── page.tsx    # Project detail view

public/
└── tracker.js          # Self-hosted analytics tracker
```

## Database

### Connection

Uses `@neondatabase/serverless` - HTTP-based, no connection pool needed:

```typescript
import { neon } from '@neondatabase/serverless';
const sql = neon(process.env.DATABASE_URL!);

// Query example
const result = await sql`SELECT * FROM projects`;
```

### Schema Management

Schema is defined in `lib/schema.sql`. To apply changes:

```bash
npm run db:push
# or manually:
psql $DATABASE_URL -f lib/schema.sql
```

### Key Tables

- `projects` - Project metadata (4 tracked projects)
- `events` - Analytics events (partitioned by quarter)
- `daily_stats` - Aggregated daily statistics
- `v_project_overview` - Dashboard view

## Development Patterns

### Server Components (Default)

All pages use React Server Components - data fetching happens on the server:

```typescript
// app/dashboard/page.tsx
export default async function DashboardPage() {
  const data = await sql`SELECT * FROM v_project_overview`;
  return <div>{/* render data */}</div>;
}
```

### Dynamic Routes

Project detail pages use Next.js dynamic segments:

```typescript
// app/dashboard/[project]/page.tsx
interface PageProps {
  params: Promise<{ project: string }>;
  searchParams: Promise<{ range?: string }>;
}

export default async function ProjectDashboard({ params, searchParams }: PageProps) {
  const { project } = await params;
  const { range = '7' } = await searchParams;
  // ...
}
```

### API Routes

Event ingestion uses Next.js Route Handlers:

```typescript
// app/api/ingest/route.ts
export async function POST(request: NextRequest) {
  // Verify signature, parse body, insert to DB
}

export async function GET() {
  return NextResponse.json({ status: 'ok' });
}
```

## Configuration

### Next.js (`next.config.ts`)

- Image optimization for Google Favicons
- Security headers (X-Frame-Options, CSP, etc.)
- CORS for Vercel ingest endpoint

### TypeScript (`tsconfig.json`)

- Strict mode enabled
- Path alias: `@/*` → project root
- Next.js plugin for App Router types

### Tailwind (`tailwind.config.ts`)

- Default configuration
- Content paths: `app/**/*.{ts,tsx}`

## Deployment

### Vercel (Recommended)

1. Connect repository to Vercel
2. Set environment variables in Vercel dashboard
3. Deploy (automatic on push to main)

### Manual Build

```bash
npm run build
npm run start
```

## Troubleshooting

### Common Issues

| Issue | Solution |
|-------|----------|
| DB connection fails | Check `DATABASE_URL` format, ensure NeonDB is active |
| Events not received | Verify `DRAIN_SECRET` matches Vercel Drain settings |
| Build error | Check TypeScript errors with `npm run build` |
| No data in dashboard | Check if events table has data, verify project mapping |

### Useful Commands

```bash
# Check database connection
psql $DATABASE_URL -c "SELECT COUNT(*) FROM events"

# View recent events
psql $DATABASE_URL -c "SELECT * FROM events ORDER BY timestamp DESC LIMIT 5"

# Check project overview
psql $DATABASE_URL -c "SELECT * FROM v_project_overview"
```

## tracker.js Integration

### Adding tracker.js to a Project

Include the script in your project's `<head>`:

```html
<script
  defer
  data-project="your-project-id"
  src="https://analytics.masem.at/tracker.js"
></script>
```

**Options:**
- `data-project` - Project ID (optional, auto-detects from hostname)
- `data-api` - Custom API endpoint (default: `https://analytics.masem.at/api/events`)

### Custom Event Tracking

```typescript
// Track custom events
window.masemit?.track('button_click', {
  button: 'signup',
  location: 'header'
});
```

### Owner Exclusion

To exclude yourself from tracking:
- Add `?owner=sempre` to any page URL
- Or visit any `/admin` path on the tracked site

This sets a localStorage flag that persists across sessions.

## Known Issues

None currently.

## Completed Features (v0.2.0)

- [x] Self-hosted tracker.js (replaces Vercel Analytics)
- [x] Country map statistics (geo visualization)
- [x] Settings page for project management
- [x] Custom date range filtering

## Planned Features

- [ ] Phase out Vercel Drains completely
- [ ] Add authentication for dashboard access
- [ ] Export data to CSV
