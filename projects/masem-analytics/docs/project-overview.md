# Project Overview - masemIT Analytics

> Internal unified analytics dashboard for masemIT projects

## Quick Reference

| Attribute | Value |
|-----------|-------|
| **Name** | masemIT Analytics |
| **Type** | Internal Dashboard |
| **Status** | 🔴 Build Error (in progress) |
| **Owner** | masemIT e.U. |
| **Repository** | monolith |

## Purpose

Aggregates web analytics from all masemIT projects into a single dashboard, powered by Vercel Web Analytics Drains.

### Projects Tracked

| Project | Domain | Icon | Color |
|---------|--------|------|-------|
| masem.at | www.masem.at | 🏠 | #0F2C55 |
| ChainSights | chainsights.one | ⛓️ | #00E5C0 |
| tellingCube | tellingcube.com | 📊 | #B84EFF |
| hoki.help | hoki.help | 💝 | #FF6B6B |

## Tech Stack Summary

| Layer | Technology |
|-------|------------|
| Framework | Next.js 16.1.4 (App Router) |
| Language | TypeScript 5 |
| UI | React 19 + Tailwind CSS |
| Database | NeonDB (PostgreSQL) |
| Hosting | Vercel |

## Architecture

**Serverless Web Application** using React Server Components for direct database access.

```
Vercel Drains → /api/ingest → NeonDB → Dashboard (Server Components)
```

## Key Features

- **Unified View** - All 4 projects in one dashboard
- **Real-time** - Live event ingestion from Vercel Drains
- **Per-Project Analytics** - Detailed views per project
- **Time Ranges** - Today, 7d, 30d, 90d filters
- **Metrics** - Page views, visitors, referrers, countries, devices, UTM

## Documentation Index

- [Architecture](./architecture.md) - System design & patterns
- [Development Guide](./development-guide.md) - Setup & development
- [Source Tree](./source-tree-analysis.md) - Code structure
- [API Contracts](./api-contracts.md) - API documentation
- [Data Models](./data-models.md) - Database schema

## Company Context

This project is part of **masemIT e.U.** internal tooling.

See: `C:\Users\mario\Sources\dev\masemit-common\company-info.md`

## Current Status

- 🔴 **Build Error** - Needs fixing before deployment
- 🗺️ **Planned** - Country map statistics

## Getting Started

```bash
npm install
cp .env.example .env
# Edit .env with credentials
npm run db:push
npm run dev
```

## Links

- **Vercel Project** - Check Vercel Dashboard
- **NeonDB Console** - [neon.tech](https://neon.tech)
- **Company Site** - [masem.at](https://masem.at)
