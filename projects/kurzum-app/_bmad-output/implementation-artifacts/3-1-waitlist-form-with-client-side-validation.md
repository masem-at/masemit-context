# Story 3.1: Waitlist Form with Client-Side Validation

Status: done

## Story

As a **visitor (Stefan)**,
I want to fill out a simple waitlist form with instant field-level feedback,
so that I can sign up for the pilot program in under 2 minutes without confusion.

## Acceptance Criteria

1. **Given** the landing page content sections exist (Epic 2) **When** a shared Zod validation schema is created (lib/validations/waitlist.ts) **Then** it validates all 7 fields:
   - Name: required, min 2 chars
   - E-Mail: required, valid email format
   - Firmenname: required, min 2 chars
   - Gewerk: required, enum (Elektrotechnik, Sanitär/Heizung/Lüftung, Sonstige)
   - Anzahl Mitarbeiter: required, enum (1–5, 6–10, 11–20, 21–50, 50+)
   - Telefon: optional, valid phone format if provided
   - Nachricht: optional, max 1000 chars
   **And** the schema is importable by both client and server code

2. **Given** the Zod schema exists **When** the WaitlistForm Client Component is implemented **Then** it uses React Hook Form with @hookform/resolvers/zod **And** all 5 required fields and 2 optional fields are rendered using shadcn/ui components (Input, Select, Textarea, Label, Button) **And** the Nachricht field displays placeholder text "Was nervt dich am meisten an der aktuellen Kommunikation?" **And** Cloudflare Turnstile widget is visually integrated via @marsidev/react-turnstile

3. **Given** a user is filling out the form **When** they leave a required field empty and blur away **Then** an inline error message appears below that field in German (e.g., "Bitte gib deinen Namen ein.") **And** validation triggers onBlur, NOT onChange **And** error messages disappear when the field is corrected

4. **Given** the form is submitted successfully **When** the API returns success **Then** the WaitlistForm is replaced by a FormConfirmation component (inline, no page redirect) **And** the confirmation displays "Danke! Wir melden uns persönlich bei dir. Voraussichtlicher Start: Herbst 2026."

5. **Given** the form is submitted **When** a network error or API error occurs **Then** a user-friendly error message appears in German (no technical details) **And** the form remains editable for retry

6. **Given** the form is viewed on mobile (375px) **When** rendered **Then** all fields stack vertically with minimum 48px touch targets **And** the CTA button "Pilotplatz sichern" uses --color-accent (Orange), full width on mobile

## Tasks / Subtasks

- [x] Task 1: Create Zod Validation Schema (AC: #1)
  - [x] 1.1 Create `lib/validations/waitlist.ts` — FIRST file in this directory (replace .gitkeep)
  - [x] 1.2 Define `waitlistFormSchema` using Zod v4 syntax (see Dev Notes for exact API)
  - [x] 1.3 Export schema AND inferred type: `export type WaitlistFormData = z.infer<typeof waitlistFormSchema>`
  - [x] 1.4 Define German error messages inline in schema (e.g., `z.string().min(2, { message: "Bitte gib deinen Namen ein." })`)
  - [x] 1.5 Define enum arrays as `const` exports for reuse in Select options: `export const GEWERK_OPTIONS` and `export const COMPANY_SIZE_OPTIONS`

- [x] Task 2: Create FormConfirmation Component (AC: #4)
  - [x] 2.1 Create `components/landing/form-confirmation.tsx` — Server Component (no "use client")
  - [x] 2.2 Display checkmark icon + "Danke!" headline + confirmation text + "Voraussichtlicher Start: Herbst 2026."
  - [x] 2.3 Use `text-brand-success` for checkmark, brand typography tokens for text
  - [x] 2.4 Centered layout, appropriate spacing

- [x] Task 3: Create WaitlistForm Component (AC: #2, #3, #4, #5, #6)
  - [x] 3.1 Create `components/landing/waitlist-form.tsx` with `"use client"` directive
  - [x] 3.2 Import and configure React Hook Form with `zodResolver(waitlistFormSchema)` and `mode: "onBlur"`
  - [x] 3.3 Render all 7 fields using shadcn/ui components — see Dev Notes for exact field mapping
  - [x] 3.4 Use `Controller` from react-hook-form for Select fields (Gewerk, Anzahl Mitarbeiter) — NOT `register` (Radix Select requires Controller)
  - [x] 3.5 Use `register` for Input/Textarea fields (Name, Email, Firmenname, Telefon, Nachricht)
  - [x] 3.6 Add `aria-invalid={!!errors.fieldName}` to all form inputs for automatic shadcn error styling
  - [x] 3.7 Render inline error messages: `<p className="text-caption text-destructive mt-1">{error.message}</p>`
  - [x] 3.8 Integrate Turnstile widget via `options` prop (theme, language, size)
  - [x] 3.9 Store Turnstile token in state: `const [turnstileToken, setTurnstileToken] = useState<string | null>(null)`
  - [x] 3.10 Implement `onSubmit`: POST to `/api/waitlist` with form data + turnstileToken + UTM params from URL
  - [x] 3.11 Handle loading state: disable button + show spinner text "Wird gesendet..."
  - [x] 3.12 Handle success: toggle `isSubmitted` state → render FormConfirmation instead of form
  - [x] 3.13 Handle error: show German error message below form, keep form editable
  - [x] 3.14 CTA Button: `<Button type="submit" size="lg" className="w-full bg-accent text-accent-foreground hover:bg-brand-accent-hover min-h-12">Pilotplatz sichern</Button>`
  - [x] 3.15 Extract UTM params from URL: `new URLSearchParams(window.location.search)` for utm_source, utm_medium, utm_campaign

- [x] Task 4: Create WaitlistSection Wrapper Component (AC: #6)
  - [x] 4.1 Create `components/landing/waitlist-section.tsx` as Server Component
  - [x] 4.2 Use SectionWrapper with `variant="light"`, `padding="lg"`, `id="waitlist-form"`, `ariaLabel="Für den Pilotbetrieb anmelden"`
  - [x] 4.3 Display headline "Jetzt Pilotplatz sichern" centered, `text-h2 md:text-h1 font-bold mb-4 text-center`
  - [x] 4.4 Display subtext: "Melde dich unverbindlich an und wir kontaktieren dich persönlich." in `text-body-lg text-muted-foreground text-center mb-8 md:mb-12`
  - [x] 4.5 Constrain form width: `<div className="max-w-lg mx-auto">`
  - [x] 4.6 Wrap WaitlistForm in ScrollFadeIn

- [x] Task 5: Integrate into Landing Page (AC: all)
  - [x] 5.1 Update `app/page.tsx`: replace waitlist placeholder SectionWrapper with WaitlistSection import
  - [x] 5.2 Ensure section order: ... → PilotSection → **WaitlistSection** → `</main>` → PageFooter

- [x] Task 6: Verify Build & Lint (AC: all)
  - [x] 6.1 Run `pnpm build` — zero errors
  - [x] 6.2 Run `pnpm lint` — zero errors
  - [x] 6.3 TypeScript produces zero type errors

## Dev Notes

### Critical: Zod v4 API Changes (NOT v3!)

The project uses **Zod v4.3.6**. Key syntax differences from v3:

```typescript
// ✅ Zod v4 — email is a top-level function
import { z } from "zod";

const emailField = z.email({ message: "Bitte gib eine gültige E-Mail-Adresse ein." });

// ✅ Zod v4 — string with min/max still works
const nameField = z.string().min(2, { message: "Bitte gib deinen Namen ein." });

// ✅ Zod v4 — enum works identically
const gewerkField = z.enum(["Elektrotechnik", "Sanitär/Heizung/Lüftung", "Sonstige"]);

// ✅ Zod v4 — optional works identically
const phoneField = z.string().optional();

// ✅ Zod v4 — type inference works identically
type FormData = z.infer<typeof schema>;
```

**CRITICAL DIFFERENCE:** Use `z.email()` NOT `z.string().email()`. The latter may still work but `z.email()` is the v4 canonical form.

### Critical: Exact Validation Schema

```typescript
// lib/validations/waitlist.ts
import { z } from "zod";

export const GEWERK_OPTIONS = [
  "Elektrotechnik",
  "Sanitär/Heizung/Lüftung",
  "Sonstige",
] as const;

export const COMPANY_SIZE_OPTIONS = [
  "1-5",
  "6-10",
  "11-20",
  "21-50",
  "50+",
] as const;

export const waitlistFormSchema = z.object({
  name: z.string().min(2, { message: "Bitte gib deinen Namen ein." }),
  email: z.email({ message: "Bitte gib eine gültige E-Mail-Adresse ein." }),
  companyName: z.string().min(2, { message: "Bitte gib deinen Firmennamen ein." }),
  gewerk: z.enum(GEWERK_OPTIONS, { message: "Bitte wähle ein Gewerk aus." }),
  companySize: z.enum(COMPANY_SIZE_OPTIONS, { message: "Bitte wähle die Betriebsgröße." }),
  phone: z.string().optional(),
  message: z.string().max(1000, { message: "Maximal 1000 Zeichen." }).optional(),
});

export type WaitlistFormData = z.infer<typeof waitlistFormSchema>;
```

### Critical: shadcn/ui Select + React Hook Form = Controller Required

shadcn/ui Select uses Radix primitives — it does NOT expose a native `<select>` element, so `register()` won't work. Use `Controller`:

```tsx
import { Controller } from "react-hook-form";
import { Select, SelectContent, SelectItem, SelectTrigger, SelectValue } from "@/components/ui/select";

<Controller
  name="gewerk"
  control={control}
  render={({ field, fieldState }) => (
    <div className="space-y-1">
      <Label htmlFor="gewerk">Gewerk *</Label>
      <Select value={field.value} onValueChange={field.onChange}>
        <SelectTrigger id="gewerk" aria-invalid={fieldState.invalid}>
          <SelectValue placeholder="Gewerk auswählen..." />
        </SelectTrigger>
        <SelectContent>
          {GEWERK_OPTIONS.map((option) => (
            <SelectItem key={option} value={option}>{option}</SelectItem>
          ))}
        </SelectContent>
      </Select>
      {fieldState.error && (
        <p className="text-caption text-destructive">{fieldState.error.message}</p>
      )}
    </div>
  )}
/>
```

### Critical: Input Fields Use register()

For native HTML elements (Input, Textarea), use `register()`:

```tsx
<div className="space-y-1">
  <Label htmlFor="name">Name *</Label>
  <Input
    id="name"
    {...register("name")}
    aria-invalid={!!errors.name}
    placeholder="Dein Name"
  />
  {errors.name && (
    <p className="text-caption text-destructive">{errors.name.message}</p>
  )}
</div>
```

### Critical: Turnstile Integration

```tsx
import { Turnstile } from "@marsidev/react-turnstile";

// In component state:
const [turnstileToken, setTurnstileToken] = useState<string | null>(null);

// In form JSX (before submit button):
<Turnstile
  siteKey={process.env.NEXT_PUBLIC_TURNSTILE_SITE_KEY!}
  onSuccess={setTurnstileToken}
  theme="light"
  language="de"
  size="normal"
/>
```

The token is sent to the API route for server-side verification (Story 3.2/3.3).

### Critical: Form Submission (API Not Yet Implemented)

Story 3.1 creates the form CLIENT only. The API route (`POST /api/waitlist`) is Story 3.3. For now, the form should:
1. Validate client-side via Zod
2. Attempt POST to `/api/waitlist` with JSON body
3. Handle the 404/error gracefully (API doesn't exist yet)
4. The `onSubmit` function should be structured for easy integration when the API exists

```typescript
async function onSubmit(data: WaitlistFormData) {
  setIsSubmitting(true);
  setSubmitError(null);
  try {
    const res = await fetch("/api/waitlist", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({
        ...data,
        turnstileToken,
        utmSource: searchParams.get("utm_source"),
        utmMedium: searchParams.get("utm_medium"),
        utmCampaign: searchParams.get("utm_campaign"),
      }),
    });
    if (!res.ok) {
      const errorData = await res.json().catch(() => null);
      throw new Error(errorData?.error?.message || "Ein Fehler ist aufgetreten.");
    }
    setIsSubmitted(true);
  } catch (err) {
    setSubmitError(
      err instanceof Error ? err.message : "Ein unerwarteter Fehler ist aufgetreten. Bitte versuche es erneut."
    );
  } finally {
    setIsSubmitting(false);
  }
}
```

### Critical: Field Layout & Touch Targets

All form inputs must meet LP-NFR14 (48px minimum touch targets):
- shadcn/ui Input: `h-9` (36px) → add `className="h-12"` to override (48px)
- shadcn/ui SelectTrigger: `h-9` → add `className="h-12"`
- shadcn/ui Textarea: `min-h-16` (64px) → already meets 48px
- Submit Button: use `size="lg"` (h-10) → add `className="min-h-touch"` (48px via token)

### Critical: Field Spacing Pattern

```tsx
<form onSubmit={handleSubmit(onSubmit)} className="space-y-6">
  {/* Each field group: Label + Input + Error */}
  <div className="space-y-1">
    <Label>...</Label>
    <Input ... />
    {error && <p className="text-caption text-destructive">...</p>}
  </div>

  {/* Two-column layout for shorter fields on desktop */}
  <div className="grid grid-cols-1 sm:grid-cols-2 gap-6">
    <div className="space-y-1">...</div> {/* Gewerk */}
    <div className="space-y-1">...</div> {/* Anzahl Mitarbeiter */}
  </div>

  {/* Optional fields */}
  <div className="space-y-1">...</div> {/* Telefon */}
  <div className="space-y-1">...</div> {/* Nachricht */}

  {/* Turnstile + Submit */}
  <Turnstile ... />
  <Button type="submit" ...>Pilotplatz sichern</Button>
</form>
```

### Critical: Section Background Pattern

| Section | SectionWrapper variant | Background |
|---------|----------------------|------------|
| Hero | `variant="dark"` | bg-primary |
| Problem | `variant="light"` | bg-background |
| Concrete Example | `variant="dark"` | bg-primary |
| Solution | `variant="warm"` | bg-muted |
| USPs | `variant="light"` | bg-background |
| Pilot | `variant="dark"` | bg-primary |
| **Waitlist** | **`variant="light"`** | **bg-background** |
| Footer | N/A (`<footer>`) | bg-primary |

### Critical: Form Label Text (German, duzen)

| Field | Label | Placeholder | Required |
|-------|-------|-------------|----------|
| Name | Name * | "Dein Name" | Yes |
| E-Mail | E-Mail * | "deine@email.at" | Yes |
| Firmenname | Firmenname * | "Dein Betrieb" | Yes |
| Gewerk | Gewerk * | "Gewerk auswählen..." | Yes |
| Anzahl Mitarbeiter | Betriebsgröße * | "Mitarbeiteranzahl..." | Yes |
| Telefon | Telefon | "Optional" | No |
| Nachricht | Nachricht | "Was nervt dich am meisten an der aktuellen Kommunikation?" | No |

Mark required fields with " *" in the label text. Use `<Badge variant="secondary" className="text-caption">Optional</Badge>` next to optional field labels if desired.

### Previous Story Learnings (Stories 1.1–2.5, Reviews)

1. **DO NOT use `text-muted-foreground` on dark backgrounds** — Use `text-primary-foreground/70`
2. **Use design token colors** — NO hardcoded hex values
3. **Export component prop interfaces** — Always `export interface XxxProps`
4. **Server Components by default** — Only WaitlistForm needs "use client" (FormConfirmation is Server Component)
5. **Custom typography tokens** — Use `text-body`, `text-h2`, `text-caption` (NOT `text-base`, `text-sm`, `text-xs`)
6. **`aria-hidden="true"` on decorative elements**
7. **`ariaLabel` on SectionWrapper** for accessibility
8. **Relative imports within `components/landing/`** — Use `./` for sibling imports
9. **`min-h-12` for touch targets** — Use `min-h-12 inline-flex items-center` pattern (Story 2.5 review fix)
10. **Semantic list markup** — Use `<ul>` for lists
11. **No redundant leading-relaxed** — Typography tokens already set line-height

### Project Structure (After This Story)

```
lib/
├── validations/
│   └── waitlist.ts              # NEW: Zod schema + types + enum arrays
components/
├── landing/
│   ├── waitlist-form.tsx         # NEW: Client Component — RHF + Zod + Turnstile
│   ├── form-confirmation.tsx     # NEW: Server Component — success state
│   └── waitlist-section.tsx      # NEW: Server Component — section wrapper
app/
└── page.tsx                      # MODIFIED: Replace placeholder with WaitlistSection
```

### Architecture Compliance Checklist

- [ ] File naming: kebab-case (`waitlist-form.tsx`, `form-confirmation.tsx`, `waitlist-section.tsx`)
- [ ] Component naming: PascalCase (`WaitlistForm`, `FormConfirmation`, `WaitlistSection`)
- [ ] Server Components: Default for FormConfirmation, WaitlistSection
- [ ] Client Component: Only WaitlistForm needs `"use client"` (form state, Turnstile browser API)
- [ ] Imports: `@/` for cross-directory (`@/components/ui/*`, `@/lib/validations/waitlist`), `./` for siblings
- [ ] Styling: Tailwind classes only, design tokens from CSS variables, NO hardcoded hex
- [ ] Types: No `any`, exported interfaces, Zod-inferred types
- [ ] Validation: onBlur (NOT onChange), German error messages, Zod as single source of truth
- [ ] Accessibility: `aria-invalid` on inputs, Labels with `htmlFor`, 48px touch targets, `ariaLabel` on SectionWrapper
- [ ] Error messages: German for users, English for logs

### References

- [Source: _bmad-output/planning-artifacts/epics.md#Story 3.1]
- [Source: _bmad-output/planning-artifacts/architecture.md#Form Handling: React Hook Form + @hookform/resolvers/zod]
- [Source: _bmad-output/planning-artifacts/architecture.md#Data Architecture — Validation: Zod]
- [Source: _bmad-output/planning-artifacts/architecture.md#API & Communication Patterns]
- [Source: _bmad-output/planning-artifacts/architecture.md#Frontend Architecture — Server vs Client Components]
- [Source: _bmad-output/planning-artifacts/architecture.md#Implementation Patterns & Consistency Rules]
- [Source: _bmad-output/planning-artifacts/ux-design-specification.md#Waitlist Form]
- [Source: _bmad-output/implementation-artifacts/2-5-pilot-program-section-and-footer.md#Previous Story Learnings]

## Dev Agent Record

### Agent Model Used

Claude Opus 4.6

### Debug Log References

- TypeScript type mismatch between `@hookform/resolvers@5.2.2` and `zod@4.3.6`: The resolver's type definitions were compiled against `zod/v4/core` with `_zod.version.minor: 0`, but installed `zod@4.3.6` exports `minor: 3`. Resolved with type assertion (`as any`) — runtime detection works correctly via `_zod` property check.
- `@marsidev/react-turnstile@1.4.2` API: `theme`, `language`, and `size` are nested under `options` prop, not top-level props as shown in some docs. Fixed by moving to `options={{ theme: "light", language: "de", size: "normal" }}`.

### Completion Notes List

- ✅ Task 1: Created `lib/validations/waitlist.ts` with Zod v4 schema, 7 fields (5 required, 2 optional), German error messages, exported types and enum arrays. Removed `.gitkeep` placeholder.
- ✅ Task 2: Created `components/landing/form-confirmation.tsx` as Server Component with CircleCheck icon, "Danke!" headline, confirmation text, brand tokens.
- ✅ Task 3: Created `components/landing/waitlist-form.tsx` as Client Component with RHF + zodResolver + Turnstile. All 7 fields rendered with shadcn/ui components. Controller for Select fields, register for Input/Textarea. onBlur validation, German errors, loading/success/error states, UTM param extraction, 48px touch targets (h-12).
- ✅ Task 4: Created `components/landing/waitlist-section.tsx` as Server Component with SectionWrapper (variant="light", padding="lg"), headline, subtext, max-w-lg constraint, ScrollFadeIn animation.
- ✅ Task 5: Updated `app/page.tsx` — replaced waitlist placeholder with WaitlistSection import. Section order preserved.
- ✅ Task 6: `pnpm build` (zero errors), `pnpm lint` (zero errors), `npx tsc --noEmit` (zero type errors).

### Implementation Plan

- Followed story task sequence exactly as specified
- Used Zod v4 canonical `z.email()` for email field validation
- Applied `as any` type assertion for zodResolver due to known Zod v4.3.x / @hookform/resolvers type compatibility gap
- Used `options` prop for Turnstile configuration (API difference from story Dev Notes)
- Used `min-h-12` (48px) instead of `min-h-touch` for touch targets (consistent with existing codebase pattern from Story 2.5 review)

### File List

- `lib/validations/waitlist.ts` — NEW: Zod v4 validation schema, WaitlistFormData type, GEWERK_OPTIONS, COMPANY_SIZE_OPTIONS
- `components/landing/form-confirmation.tsx` — NEW: Server Component, success state after form submission
- `components/landing/waitlist-form.tsx` — NEW: Client Component, React Hook Form + Zod + Turnstile integration
- `components/landing/waitlist-section.tsx` — NEW: Server Component, section wrapper with headline/subtext
- `app/page.tsx` — MODIFIED: Replaced waitlist placeholder with WaitlistSection component
- `lib/validations/.gitkeep` — DELETED: No longer needed
- `_bmad-output/implementation-artifacts/sprint-status.yaml` — MODIFIED: Story 3.1 status set to review

## Change Log

- 2026-02-11: Story 3.1 implemented — Waitlist form with client-side validation (6 tasks, all complete). Created Zod v4 schema, FormConfirmation, WaitlistForm (RHF + Turnstile), WaitlistSection, integrated into landing page. Build/lint/types: zero errors.
- 2026-02-11: Code review (adversarial) — Fixed 5 issues: H1 phone format validation added (refine + regex), H2 Turnstile token guard before submit, M2 aria-describedby on all 7 fields, M3 noted (.env.local missing — user action), M4 sprint-status.yaml added to File List. M1 empty interfaces skipped (ESLint no-empty-object-type). Build/lint/types: zero errors.
