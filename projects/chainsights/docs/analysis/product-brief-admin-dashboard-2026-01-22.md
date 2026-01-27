---
stepsCompleted: [1, 2, 3, 4, 5]
inputDocuments:
  - docs/analysis/research/market-web3-analytics-europe-research-2025-12-15.md
workflowType: 'product-brief'
lastStep: 5
project_name: 'chainsights-admin-dashboard'
user_name: 'Mario'
date: '2026-01-22'
feature_context: 'Admin Dashboard Redesign with UX improvements'
---

# Product Brief: ChainSights Admin Dashboard Redesign

**Date:** 2026-01-22
**Author:** Mario

---

## Prior Research Context

### UX Research Findings (Session 2026-01-22)

Key patterns identified for admin dashboard redesign:

1. **Semantic Grouping** - Group by function (Actions, Content Management, Preview/Display)
2. **Stratified Layout** - Top-down priority (status → actions → details)
3. **Tabs/Sidebar Navigation** - Vercel/GitHub style parallel navigation
4. **Card-Based Action Grouping** - Clear visual boundaries with shadcn/ui cards
5. **Progressive Disclosure** - Collapse detailed views, expand on demand
6. **Quick Actions Menu** - Command palette (⌘K) for power users
7. **Status Overview** - Glanceable "is everything OK?" section

**Current State Issues:**
- Mixed components on single page: recalculate, test report, feature list, PDF display, source display
- No clear organizational structure
- New feature needs to be integrated

**Benchmark References:**
- Vercel Dashboard (left sidebar, command menu)
- GitHub Dashboard (semantic organization)

---

## Executive Summary

ChainSights Admin Dashboard Redesign transforms a cluttered single-page admin interface into a structured, navigable dashboard optimized for daily use. The redesign prioritizes instant status visibility ("is everything OK?"), clear navigation to functional areas, and logical grouping of related features.

---

## Core Vision

### Problem Statement

The current ChainSights admin page has grown organically into a cluttered interface where recalculation triggers, test reports, feature management, PDF displays, and source displays all compete for attention on a single page. For a daily-use tool, this creates friction in finding and executing common tasks.

### Problem Impact

- **Time waste**: Scanning through unrelated components to find needed functions
- **Cognitive load**: No visual hierarchy means every element demands equal attention
- **Missing context**: No "at a glance" status indicators for system health
- **Scaling issues**: Adding new features only increases clutter

### Why Existing Solutions Fall Short

The current implementation lacks:
1. **Navigation structure** — No sidebar or tabs to organize functional areas
2. **Status overview** — No key indicators (last runs, failures, health)
3. **Semantic grouping** — Actions, content management, and displays are intermixed
4. **Progressive disclosure** — Everything is visible all the time

### Proposed Solution

A structured admin dashboard following Vercel/GitHub UX patterns:

1. **Sidebar navigation** for clear section switching
2. **Status overview panel** showing key indicators at top
3. **Card-based groupings** for related functions
4. **Progressive disclosure** for detailed views (PDF, source)
5. **Command menu (⌘K)** for power-user quick access

### Key Differentiators

- **Daily-use optimized**: Designed for the actual admin workflow, not feature accumulation
- **Glanceable status**: Instantly know if everything is healthy
- **Familiar patterns**: Follows Vercel/GitHub conventions (low learning curve)
- **Scalable structure**: New features fit naturally into defined sections

---

## Target Users

### Primary Users

**Persona: Mario (Solo Admin/Founder)**

| Attribute | Details |
|-----------|---------|
| **Role** | ChainSights founder & sole administrator |
| **Usage** | Daily |
| **Technical Level** | Expert (built the system) |
| **Environment** | Development machine, browser-based admin |

**Primary Workflows:**

1. **Report Generation Flow**
   - Trigger: Customer report request arrives
   - Need: Quickly generate report for specific DAO
   - Current pain: Hunting through cluttered page for report tools

2. **Health Check Flow**
   - Trigger: Daily routine / after deployments
   - Need: Verify system health (last runs, failures, job status)
   - Current pain: No "at a glance" status indicators

**What Success Looks Like:**
> "I open the admin, instantly see if everything is healthy, and can generate a report in 3 clicks or less."

### Secondary Users

N/A — Single admin system. No secondary users planned.

### User Journey

| Stage | Mario's Experience |
|-------|-------------------|
| **Open Admin** | See status overview immediately (healthy/issues) |
| **Health Check** | Glance at key indicators, done in 5 seconds |
| **Report Request** | Navigate to Reports section → Select DAO → Generate |
| **Deep Dive** | Only when needed: view logs, PDF preview, source data |

**Critical Success Moment:** Opening the dashboard and knowing within 2 seconds if action is needed.

---

## Success Metrics

### User Success Metrics

| Metric | Current State | Target | How to Measure |
|--------|--------------|--------|----------------|
| **Time to Status Check** | Unknown (requires scanning) | < 2 seconds | Glanceable status panel at top |
| **Clicks to Report** | Unknown (searching required) | ≤ 3 clicks | Admin → Reports → Generate |
| **Find Any Function** | "Always searching where is what" | ≤ 2 clicks from sidebar | Clear navigation structure |

### Qualitative Success Criteria

The redesign is successful when opening the admin feels like:
- "Clean, everything in its place"
- "I know exactly where to go"
- "I can do my task and close it quickly"

### Business Objectives

| Objective | Rationale |
|-----------|-----------|
| **Reduce Admin Friction** | Daily tool should not slow down operations |
| **Enable Scaling** | New features fit naturally without adding clutter |
| **Maintainable Structure** | Clear sections make future changes easier |

### Key Performance Indicators

1. **Status Visibility** — Health indicators visible without scrolling: ✅/❌
2. **Navigation Clarity** — All major sections accessible from sidebar: ✅/❌
3. **Task Efficiency** — Common tasks (report, recalc) complete in ≤ 3 actions: ✅/❌
4. **Zero Search** — No need to scan/search for functions: ✅/❌

*Note: For a single-user internal tool, success is measured by subjective satisfaction + objective task efficiency, not traditional business KPIs.*

---

## MVP Scope

### Core Features

| # | Feature | Description | Success Criteria |
|---|---------|-------------|------------------|
| 1 | **Sidebar Navigation** | Left sidebar with clear section links (Overview, Reports, Actions, Content) | All sections reachable in ≤2 clicks |
| 2 | **Status Overview Panel** | Top panel showing key health indicators (last GVS run, failures, job status) | Health visible in <2 seconds without scrolling |
| 3 | **Reports Section** | Dedicated area for report generation and management | Generate report in ≤3 clicks |
| 4 | **Actions Section** | Grouped triggers: Recalculate GVS, Test Report, etc. | Clear card-based action grouping |
| 5 | **Content Section** | Feature list management, Featured DAOs table | Logical grouping of content management |

### Out of Scope for MVP

| Feature | Rationale | Target Phase |
|---------|-----------|--------------|
| Command Menu (⌘K) | Power-user enhancement, not blocking daily work | v2.0 |
| Progressive Disclosure | UI polish, core navigation solves the problem | v2.0 |
| PDF/Source Preview Panels | Move to separate routes or modals | v2.0 |
| Multi-user support | Single admin for now | Future |

### MVP Success Criteria

The MVP is successful when:

1. **Status Check** — Health indicators visible without scrolling ✅
2. **Navigation** — All sections accessible from sidebar ✅
3. **Task Efficiency** — Report generation completes in ≤3 clicks ✅
4. **Zero Search** — No scanning/searching required to find functions ✅
5. **Subjective** — "Clean, everything in its place" feeling achieved ✅

### Future Vision

**v2.0 Enhancements:**
- Command palette (⌘K) for keyboard-first navigation
- Collapsible sections for progressive disclosure
- Dedicated Preview panel with slide-over for PDF/source
- Activity log / audit trail

**Long-term:**
- Multi-user support with role-based access
- Customizable dashboard widgets
- API health monitoring integration

