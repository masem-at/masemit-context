# DEV TICKET: Floating Feedback Bubble

**Priority:** Medium-High
**Effort:** Small
**Dependencies:** None (standalone component)

---

## Context

ChainSights hat aktuell keine Möglichkeit für Besucher, schnelles Feedback zu geben. Kein Survey, kein Kontaktformular, keine Feedback-Box. Bei 0 Conversions und 0 Rückmeldungen fehlt jede Insight darüber, was Besucher denken, brauchen oder vermissen.

Klassische Lösungen (Hotjar Surveys, Intercom, Typeform-Popups) sind zu invasiv für die kleine Besucherzahl und den Web3-Space, wo Privacy und Minimalismus geschätzt werden.

**Lösung:** Ein kleiner, unaufdringlicher Feedback-Kreis der auf jeder Seite schwebt. Ein Klick expandiert ihn zu einer einzeiligen Textbox. Anonym, schnell, fertig.

---

## UX Concept

### State 1: Collapsed (Default)

```
                                              ┌─────┐
                                              │ 💬  │
    [... Page Content ...]                    └─────┘
                                            bottom-right
                                            floating circle
                                            ~48px diameter
```

- Kleiner Kreis, bottom-right, fixed position
- Dezent aber sichtbar (ChainSights accent color #009BB1, leichter Puls oder Glow)
- Icon: 💬 oder custom Speech-Bubble-Icon
- Tooltip on hover: "Share your thoughts"
- Kein Text außen, nur das Icon — clean und nicht invasiv

### State 2: Expanding (Animation)

```
    ┌──────────────────────────────────────────────────────────────┐
    │ 💬  What's missing? What would make this useful for you?  ✓ ✕│
    └──────────────────────────────────────────────────────────────┘
```

- Klick auf Kreis → animiert expandieren zu voller Breite (bottom of viewport)
- Smooth transition: circle morphs → rounded rectangle → full-width bar
- Duration: ~300ms ease-out
- Textinput autofocused nach Animation

### State 3: Active Input

```
    ┌──────────────────────────────────────────────────────────────┐
    │ 💬  [I wish I could compare two DAOs side by side...   ] ✓ ✕│
    └──────────────────────────────────────────────────────────────┘
```

- Placeholder text (rotierend, siehe unten)
- Enter oder ✓ = Senden
- Escape oder ✕ = Abbrechen (collapse zurück zum Kreis)
- Character limit: 500 Zeichen
- Kein Name, keine Email — 100% anonym

### State 4: Sent Confirmation

```
    ┌──────────────────────────────────────────────────────────────┐
    │ ✨  Thanks! Your feedback shapes what we build next.         │
    └──────────────────────────────────────────────────────────────┘
```

- Kurze Danke-Message (2s sichtbar)
- Auto-collapse zurück zum Kreis
- Kreis zeigt kurz ✓ statt 💬, dann zurück zu normal

---

## Rotating Placeholder Texts

Jede Page-Load zeigt einen zufälligen Placeholder:

```typescript
const placeholders = [
  "What's missing? What would make this useful for you?",
  "If you could change one thing about this page...",
  "What brought you here today?",
  "What governance data do you wish you had?",
  "Anything confusing? We'd love to know.",
  "What would make you come back tomorrow?",
  "One feature you'd pay for?",
  "What's your biggest governance headache?",
];
```

Die Placeholder geben dem User Richtung, ohne zu limitieren. Jeder davon ist auch eine Mini-Research-Frage.

---

## Page-Specific Placeholders (Optional Enhancement)

```typescript
const pageSpecificPlaceholders: Record<string, string[]> = {
  "/": [
    "What would make you try a governance report?",
    "What's missing from this page?",
  ],
  "/governance-index": [
    "Is your DAO missing from the index?",
    "What metric matters most to you?",
  ],
  "/rankings": [
    "Which DAO would you like to see here?",
    "How do you use governance rankings?",
  ],
  "/guide": [
    "Was this guide helpful? What's unclear?",
  ],
};
```

---

## Data Model

### Option A: Database (recommended)

```sql
CREATE TABLE feedback (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  message TEXT NOT NULL,
  page VARCHAR(200) NOT NULL,          -- which page it was submitted from
  referrer VARCHAR(500),               -- how they got to the site
  country VARCHAR(5),                  -- from Vercel headers (geo)
  device VARCHAR(20),                  -- 'mobile' | 'desktop' | 'tablet'
  placeholder_shown VARCHAR(200),      -- which placeholder was displayed
  created_at TIMESTAMP DEFAULT NOW()
);

-- NO user identification. No email, no IP, no cookies. Fully anonymous.
```

### Option B: Email (simpler, no DB migration)

```typescript
// Send via Resend API
await resend.emails.send({
  from: "feedback@chainsights.one",
  to: "contact@masem.at",
  subject: `[Feedback] ${page}`,
  text: `
    Message: ${message}
    Page: ${page}
    Device: ${device}
    Country: ${country}
    Placeholder: ${placeholderShown}
    Time: ${new Date().toISOString()}
  `,
});
```

### Empfehlung

**Start mit Option A (Database).** Dann kannst du später ein Admin-Dashboard bauen das alle Feedback-Einträge zeigt, filtern nach Page, sortieren nach Datum. Bei Email verlierst du Struktur.

Option B als Fallback oder zusätzliche Benachrichtigung: Email-Alert bei jedem neuen Feedback, damit du es sofort siehst.

---

## API Route

```typescript
// POST /api/feedback

interface FeedbackRequest {
  message: string;         // required, max 500 chars
  page: string;            // auto-captured from window.location.pathname
  placeholderShown: string; // which placeholder was displayed
}

// Response
// 200: { success: true }
// 400: { error: "Message required" }
// 429: { error: "Too many submissions" }
```

### Rate Limiting

- Max 3 submissions per session (sessionStorage counter)
- After 3: Bubble hides for this session
- Server-side: Max 10 per IP per hour (via Upstash Redis or simple in-memory)
- Cloudflare Turnstile NOT needed (too much friction for 1 text field)

---

## Analytics Events

```typescript
// Track feedback interactions
track("feedback_bubble_click", { page });           // Bubble opened
track("feedback_submitted", { page, length });       // Feedback sent
track("feedback_dismissed", { page });               // User closed without sending
```

---

## Design Specs

### Collapsed Bubble

| Property | Value |
|----------|-------|
| Size | 48px diameter |
| Color | #009BB1 (ChainSights accent) |
| Position | Fixed, bottom: 24px, right: 24px |
| Icon | Speech bubble (Lucide MessageCircle or custom) |
| Shadow | subtle drop shadow |
| Animation | Gentle pulse every 10s (subtle, nicht nervend) |
| Z-index | 1000 (above content, below modals) |
| Mobile | Same position, same size |

### Expanded Bar

| Property | Value |
|----------|-------|
| Width | calc(100% - 48px) with 24px margin |
| Height | ~56px |
| Position | Fixed, bottom: 24px |
| Background | #0A2E31 (dark, matches ChainSights) |
| Border | 1px solid #009BB1 |
| Border-radius | 28px |
| Input | Transparent background, white text, no border |
| Buttons | ✓ (#009BB1) and ✕ (gray) |

### Animation

| Transition | Detail |
|------------|--------|
| Expand | circle → bar, 300ms ease-out, origin bottom-right |
| Collapse | bar → circle, 200ms ease-in |
| Confirmation | fade text swap, 150ms |

---

## Acceptance Criteria

- [ ] Feedback bubble visible on every page (except /admin/*)
- [ ] Click expands to full-width input bar with smooth animation
- [ ] Placeholder text rotates randomly per page load
- [ ] Enter or ✓ submits feedback
- [ ] Escape or ✕ collapses back to bubble
- [ ] Success confirmation shown for 2 seconds
- [ ] Feedback stored in database with page context
- [ ] Email notification sent on new feedback (optional)
- [ ] Rate limited: max 3 per session, 10 per IP/hour
- [ ] Analytics events tracked (open, submit, dismiss)
- [ ] Anonymous: no user identification whatsoever
- [ ] Responsive: works on mobile and desktop
- [ ] Does not overlap with other fixed elements
- [ ] Character limit 500 with visual indicator

---

## Files to Create/Touch

1. DB migration — create `feedback` table
2. `src/components/FeedbackBubble.tsx` — Client component
3. `src/app/api/feedback/route.ts` — API endpoint
4. `src/app/layout.tsx` — Include FeedbackBubble globally
5. Optional: Email notification via Resend

---

## Out of Scope

- User identification / login required feedback
- File attachments
- Feedback categories / tags (let the text speak)
- Admin dashboard for feedback (future, when volume justifies it)
- A/B testing of placeholders (manual rotation is fine for now)
- Multi-step surveys
