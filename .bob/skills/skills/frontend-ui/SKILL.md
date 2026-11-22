---
name: frontend-ui
description: Use when the user wants to build, design, or refine frontend UI — components, pages, layouts, or design systems. Enforces production-grade aesthetics, accessibility, and code quality; prevents generic AI-slop output.
---

# Frontend UI Engineering

Apply this skill for any frontend UI/UX task. Follow the three-step loop below — do not skip the Plan or Critique steps.

---

## Step 1 — Plan (before any code)

Use `ask_followup_question` to clarify unknowns, then write a 3–5 bullet brief covering:

1. **Aesthetic direction** — commit to one before writing a line. Examples: editorial minimalism, Swiss grid, high-craft SaaS, tactile brutalism, warm editorial, dark-mode productivity. No direction = generic output.
2. **Color tokens** — name the semantic tokens you will use (`--bg-canvas`, `--bg-surface`, `--text-primary`, `--border-subtle`, `--accent-primary`). Confirm the accent and ensure WCAG AA+ contrast.
3. **Typography** — name the font pairing. At minimum: a distinctive display/heading face + a clean UI body face + monospace for data. Do not default to plain Inter everywhere.
4. **Layout approach** — grid vs. flex, breakpoints, and the spatial unit (default: 8px grid with `clamp()` for fluid sizing).
5. **Component scope** — list every interactive state that must exist: default, hover, `focus-visible`, active, disabled, loading (skeleton), empty, error.

Do not proceed to Step 2 until this brief is written.

---

## Step 2 — Build

Write the implementation following all directives below.

### Aesthetic & Anti-Generic Rules

- Commit to the aesthetic direction from Step 1. Never default to whatever "looks fine."
- **Banned patterns** (hard stops — refuse to generate these):
  - Overused purple-to-blue gradients as the primary visual treatment
  - Default centered hero with floating card grid and blob shapes in the background
  - Sterile white-card-on-white-background SaaS templates with no visual character
  - Generic "AI output" aesthetic: soft shadows everywhere, rounded-everything, pastel palette

### Typography

- Apply a deliberate font pairing — display/serif or geometric sans for headings, clean UI sans for body, monospace for code/data.
- Typographic scale: define step sizes explicitly (e.g., 12/14/16/20/24/32/48px or a modular scale).
- Headings: tight `letter-spacing` (−0.02em to −0.04em for large sizes), `line-height` ≤ 1.2.
- Body: `line-height` 1.5–1.7, max line length 60ch–75ch (`max-width: 65ch` on prose blocks).
- Tabular numerals: `font-variant-numeric: tabular-nums` on all data, metrics, prices, and timestamps.

### Color System

- Use CSS custom properties scoped to `:root` (or the framework's token system):
  ```
  --bg-canvas        (page background)
  --bg-surface       (card / panel background)
  --bg-elevated      (tooltip / popover)
  --text-primary     (body copy)
  --text-secondary   (muted labels)
  --border-subtle    (dividers, card outlines)
  --accent-primary   (CTAs, links, active states)
  --status-error     (errors)
  --status-success   (confirmations)
  ```
- 60-30-10 rule: 60% neutral canvas, 30% surface/structural, 10% accent.
- One high-contrast focal accent only. Additional hues for status only.
- Every text/background pair must pass WCAG AA (4.5:1 body, 3:1 large text).

### Layout & Spacing

- 8px spatial grid: all margin, padding, gap values are multiples of 8px (or 4px for tight sub-components).
- Use CSS Grid for page/section layout; Flexbox for component-level alignment.
- Fluid sizing with `clamp()`, `min()`, `max()` — no fixed-pixel hacks that break at odd viewports.
- Mobile-first: base styles are mobile; `@media (min-width: …)` adds complexity upward.
- No horizontal overflow on any viewport width.

### Component States

Every interactive element **must** implement all of the following before the code is considered complete:

| State | Requirement |
|---|---|
| Default | Clear visual baseline |
| Hover | Visible affordance change (color, underline, elevation shift) |
| `focus-visible` | 2px+ outline offset, never `outline: none` without a custom replacement |
| Active | Pressed/depressed feedback |
| Disabled | Reduced opacity + `cursor: not-allowed`, no pointer events |
| Loading | Skeleton screen (not a spinner alone) for content areas; button gets an inline spinner + disabled |
| Empty | Purposeful empty state with label — never a blank void |
| Error | Inline error message + field highlight using `--status-error`; never silent failure |

### Micro-Interactions

- Transitions on `transform` and `opacity` only (hardware-accelerated). Do not transition `width`, `height`, or `background-color` for performance-critical interactions.
- Duration: 120ms–200ms for hover/focus; 250ms–350ms for appearing/disappearing elements.
- Always add:
  ```css
  @media (prefers-reduced-motion: reduce) {
    *, *::before, *::after { transition-duration: 0.01ms !important; animation-duration: 0.01ms !important; }
  }
  ```

### Accessibility

- Semantic HTML5: `<main>`, `<nav>`, `<header>`, `<footer>`, `<section>`, `<article>`, `<button>`, `<a>` — never `<div>` or `<span>` for interactive elements.
- All images: meaningful `alt` text; decorative images get `alt=""`.
- Form inputs: `<label>` associated via `for`/`id` or wrapping; `aria-describedby` for hints/errors.
- Icon-only buttons: `aria-label` required.
- Keyboard navigation: every interactive element reachable and operable without a mouse.
- Color alone must never be the sole indicator of state.

### Code Quality

- Write modular, single-responsibility components.
- Use the project's existing stack — do not introduce new dependencies unless explicitly asked.
- Keep components cleanly typed (TypeScript if the project uses it).
- No inline styles for anything that belongs in the design system; use tokens or utility classes.

---

## Step 3 — Self-Critique & Refine

Before outputting the final result, audit the generated code against this checklist. Fix every failure.

**Anti-generic audit:**
- [ ] Does the design have a recognizable, committed aesthetic direction — or does it look like a template?
- [ ] Is the banned-patterns list clean? (No purple gradients, floating-card grids, blob shapes)

**Typography audit:**
- [ ] Is there a real font pairing, or is it just Inter/system-font throughout?
- [ ] Are headings tight-tracked? Is body text in the 60–75ch range?
- [ ] Are numbers/metrics using `tabular-nums`?

**Color audit:**
- [ ] Are semantic CSS tokens used consistently (no magic hex values inline)?
- [ ] Does every text/background pair pass WCAG AA?

**Layout audit:**
- [ ] Is the 8px grid respected everywhere?
- [ ] Does layout use `clamp()`/`min()`/`max()` — no fixed-pixel breakage?
- [ ] No horizontal overflow at any width?

**States audit:**
- [ ] Every interactive element has all 8 states (default, hover, focus-visible, active, disabled, loading, empty, error)?
- [ ] `prefers-reduced-motion` guard in place?

**Accessibility audit:**
- [ ] Semantic HTML5 elements used throughout?
- [ ] All inputs have labels; all icon buttons have `aria-label`?
- [ ] Keyboard navigable?

Fix all failures, then output the final code.
