# "What's Next" Page UX Specification

**Author:** Sally (UX Expert)
**Date:** 2025-12-11
**Status:** Ready for Development
**Aligned with:** Winston (Architecture), Mary (QA), John (PM)

---

## Purpose

Show potential customers the product vision and planned features to:
1. Build confidence that tellingCube is actively developed
2. Motivate early adoption ("get in early, shape the product")
3. Answer "what else is coming?" without making date promises

---

## Page Name & URL

**Name:** "What's Next"
**URL:** `/whats-next`
**Header link:** Yes - add to navigation after "My Scenarios"

---

## Design Principles

1. **No dates** - Never promise timelines
2. **Honest** - Clear distinction between "building" vs "exploring"
3. **Visual** - Use icons and cards, not walls of text
4. **Inviting** - Encourage feedback, make users feel part of the journey
5. **Professional** - B2B appropriate, not startup-hype

---

## Page Structure

### Header Section

```
What's Next

tellingCube is actively evolving. Here's where we're headed.
Your feedback as a founding member shapes our priorities.
```

---

### Section 1: Now (Currently Live)

**Visual:** Green checkmark badges, solid cards

**Content:**
| Feature | Description |
|---------|-------------|
| Company Tiers | Finance/Large Cap, Industrials/Mid Cap, Consumer Staples/Startup |
| Sales View | Revenue by month, by product, top customers, trend analysis |
| Finance View | P&L summary, cash flow, cost breakdown, margin analysis |
| CSV Export | Download any view as CSV for Excel/Power BI |
| REST API | Programmatic access to scenario data |
| Visualizations | Charts inspired by IBCS© standards |

---

### Section 2: Next (Actively Building)

**Visual:** Blue "in progress" badges, slightly muted cards

**Content:**
| Feature | Description |
|---------|-------------|
| HR View | Headcount trends, department breakdown, payroll analysis |
| More Industries | Healthcare, Retail, Professional Services sectors |
| Enhanced Exports | Excel (.xlsx) format, PDF reports |
| Scenario Comparison | Compare multiple scenarios side-by-side |

**Note text:** "These features are in active development."

---

### Section 3: Later (On the Horizon)

**Visual:** Gray "planned" badges, outlined cards

**Content:**
| Feature | Description |
|---------|-------------|
| Controlling View | Cost centers, budget allocations, variance analysis |
| Multi-Year Scenarios | Generate 3-5 year business projections |
| Custom Parameters | Adjust growth rates, seasonality, company size |
| Raw Events Export | Access underlying event data for custom analysis |

**Note text:** "Planned features - priorities may shift based on feedback."

---

### Section 4: Vision (Long-term Direction)

**Visual:** Purple "vision" badges, subtle gradient background

**Content:**
| Direction | Description |
|-----------|-------------|
| Enterprise Features | Team workspaces, role-based access, audit logs |
| White-Label | Embed tellingCube in your own training platform |
| Integrations | Connect with Power BI, Tableau, SAP |
| Custom Templates | Create industry-specific scenario templates |
| AI Customization | Natural language scenario adjustments |

**Note text:** "Our north star - where we're ultimately heading."

---

### Section 5: Feedback CTA

```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│  💬 Shape What We Build                                     │
│                                                             │
│  As a founding member, your feedback directly influences    │
│  our priorities. What would help you most?                  │
│                                                             │
│  [Email Us Your Ideas]  →  support@tellingcube.com          │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## Visual Design

### Color Scheme (Status Badges)

| Status | Badge Color | Card Style |
|--------|-------------|------------|
| Now | Green (`#10B981`) | Solid white, green left border |
| Next | Blue (`#3B82F6`) | Solid white, blue left border |
| Later | Gray (`#6B7280`) | Outlined, gray left border |
| Vision | Purple (`#8B5CF6`) | Subtle gradient background |

### Layout

- **Desktop:** 2-column grid for feature cards within each section
- **Mobile:** Single column, stacked cards
- **Max width:** 1024px centered
- **Section spacing:** 64px between sections

### Icons

Use Lucide icons for each feature:
- Company Tiers: `Building2`
- Sales View: `TrendingUp`
- Finance View: `PieChart`
- HR View: `Users`
- CSV Export: `Download`
- API: `Code`
- And so on...

---

## Header Navigation Update

**Current:** `[Logo] tellingCube                    My Scenarios`

**New:** `[Logo] tellingCube                What's Next    My Scenarios`

Place "What's Next" before "My Scenarios" to encourage discovery.

---

## FAQ Updates

Update these FAQ answers to reference the new page:

### "What company types are available?"
**Add:** "We're actively adding more industries. See [What's Next](/whats-next) for upcoming sectors."

### "Can I export the data?"
**Add:** "More export formats coming soon - see [What's Next](/whats-next)."

---

## Component Structure

```
app/
  whats-next/
    page.tsx          ← Main page component

components/
  whats-next/
    FeatureCard.tsx   ← Reusable card component
    StatusBadge.tsx   ← Now/Next/Later/Vision badges
    FeedbackCTA.tsx   ← Bottom CTA section
```

---

## Accessibility

- All status badges have text labels (not just color)
- Cards are keyboard navigable
- Sufficient color contrast on all badges
- Screen reader friendly section headings

---

## Content Guidelines

**DO:**
- Use concrete feature names
- Keep descriptions to one sentence
- Be honest about uncertainty ("priorities may shift")

**DON'T:**
- Promise dates or timeframes
- Use vague buzzwords ("revolutionary", "game-changing")
- Overcommit on features we're not sure about

---

## Acceptance Criteria

- [ ] Page accessible at `/whats-next`
- [ ] Header navigation includes "What's Next" link
- [ ] Four sections displayed: Now, Next, Later, Vision
- [ ] Each feature has icon, title, and description
- [ ] Status badges clearly distinguish sections
- [ ] Feedback CTA with email link at bottom
- [ ] Responsive design (mobile-friendly)
- [ ] FAQ updated with links to What's Next page
- [ ] Lint passes

---

## Implementation Notes

- Content can be hardcoded (no CMS needed for PoC)
- Consider extracting feature data to a constants file for easy updates
- Page should be static (no API calls needed)

---

## Future Enhancements (Not for PoC)

- Feature voting/upvoting by users
- Email signup for specific feature notifications
- Progress indicators on "Next" items
- Public changelog/release notes

---

**Sally's Note:** This page turns "what's missing" into "what's coming" - a much better narrative for early adopters!
