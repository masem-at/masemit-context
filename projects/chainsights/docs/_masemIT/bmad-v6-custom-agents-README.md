# Custom BMAD v6 Agents - Installation Guide

## Included Agents

| Agent | Name | Role | Icon |
|-------|------|------|------|
| **Maya Marketing** | Maya Rivera | Web3 Marketing Strategist | 📣 |
| **Pixel Designer** | Pixel Chen | Digital Graphic Designer (SVG) | 🎨 |

---

## Installation

### Option 1: Custom Agents Directory (Recommended)

Place the `.agent.yaml` files in your project's custom agents directory:

```
your-project/
└── bmad/
    └── custom/
        └── agents/
            ├── maya-marketing.agent.yaml    # ← Copy here
            └── pixel-designer.agent.yaml    # ← Copy here
```

Then recompile agents:

```bash
# Via BMAD installer menu
npx bmad-method install
# Select: "Compile Agents (Quick rebuild of all agent .md files)"
```

### Option 2: Config Override Directory (Survives Updates)

For customizations that persist through BMAD updates:

```
your-project/
└── bmad/
    └── _cfg/
        └── agents/
            ├── maya-marketing.agent.yaml
            └── pixel-designer.agent.yaml
```

---

## Usage

After installation, activate agents in your IDE:

### Claude Code
```
@maya-marketing
@pixel-designer
```

### Cursor / Windsurf
Select from Agent Mode sidebar or use commands.

### Available Commands

**Maya (Marketing):**
- `/twitter-thread` - Create Twitter/X thread
- `/landing-copy` - Write landing page copy
- `/content-calendar` - Create content calendar
- `/viral-loop` - Design referral mechanics
- `/launch-campaign` - Plan product launch
- `/positioning` - Define positioning & messaging

**Pixel (Designer):**
- `/create-logo` - Design logo with variations
- `/create-icon` - Create SVG icon
- `/create-svg` - Create custom SVG graphic
- `/social-graphics` - Create social media graphics
- `/data-viz` - Create data visualization
- `/brand-colors` - Define color system
- `/design-review` - Review existing assets

---

## Workflows Included

All workflow files are included! Copy the entire `bmad-workflows` folder:

```
bmad/
└── custom/
    └── workflows/
        ├── marketing/
        │   ├── twitter-thread/
        │   │   ├── workflow.yaml
        │   │   └── instructions.md      ✅ Included
        │   ├── landing-page/
        │   │   ├── workflow.yaml
        │   │   └── instructions.md      ✅ Included
        │   ├── content-calendar/
        │   │   ├── workflow.yaml
        │   │   └── instructions.md      ✅ Included
        │   ├── viral-loop/
        │   │   ├── workflow.yaml
        │   │   └── instructions.md      ✅ Included
        │   └── positioning/
        │       ├── workflow.yaml
        │       └── instructions.md      ✅ Included
        └── design/
            ├── svg-creation/
            │   ├── workflow.yaml
            │   └── instructions.md      ✅ Included
            ├── logo-creation/
            │   ├── workflow.yaml
            │   └── instructions.md      ✅ Included
            ├── icon-design/
            │   ├── workflow.yaml
            │   └── instructions.md      ✅ Included
            └── data-visualization/
                ├── workflow.yaml
                └── instructions.md      ✅ Included
```

### Missing Workflows (Create as needed)

These are referenced in agent menus but not yet created:

- `marketing/launch-campaign/` - Product launch planning
- `design/social-graphics/` - Social media graphics
- `design/color-system/` - Brand color definition
- `design/design-review/` - Design feedback workflow

### Quick Start Without All Workflows

If a workflow file is missing, the agent will still work in "chat mode" - 
just without that specific command trigger.

---

## Customization

### Change Agent Name/Personality

Edit the `persona:` section:

```yaml
persona:
  name: "Your Custom Name"
  identity: |
    Your custom background...
  communication_style: |
    Your preferred style...
```

### Use Built-in Communication Presets

BMAD v6 includes 21 presets:

**Professional:**
- `Analytical Expert`
- `Supportive Mentor`
- `Direct Consultant`
- `Collaborative Partner`

**Fun:**
- `Pulp Superhero`
- `Film Noir Detective`
- `80s Action Hero`
- `Pirate Captain`
- etc.

Example:
```yaml
communication_style: Direct Consultant
```

---

## ChainSights Brand Integration (Pixel Agent)

The Pixel Designer agent includes ChainSights brand colors:

| Role | Hex | Usage |
|------|-----|-------|
| Primary | `#0F2C55` | Navy — headers, main elements |
| Secondary | `#4A89F3` | Azure — links, buttons |
| Accent 1 | `#00E5C0` | Aqua — highlights, CTAs |
| Accent 2 | `#B84EFF` | Violet — alerts, special |
| Light | `#F5F7FA` | Backgrounds |
| Dark | `#1A1F2E` | Text, dark sections |

---

## Compatibility

- BMAD Method v6 Alpha or higher
- Works with: Claude Code, Cursor, Windsurf, VS Code + GitHub Copilot
- Node.js 20+

---

*Created for ChainSights project - December 2025*
