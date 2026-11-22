# Worked Examples

Three walkthroughs showing how the skill is applied. Each shows the decisions, not a full
implementation. Use them to calibrate depth and order.

## Example 1: SaaS admin dashboard (Operate mode)

**Brief:** internal operations console for a logistics company. Dispatchers watch shipments, act on
exceptions, and filter by region. Dense, keyboard-friendly, used all day.

**Phase 1 — Discover.** Stack from `package.json`: Next.js App Router, Tailwind v4, shadcn/ui on Radix.
Existing tokens in `app/globals.css`. Reuse the existing Button, Table and Dialog. Add a `DataTable`
composition only if the existing table cannot sort and filter.

**Phase 2 — Direction.** Mode: Operate. Density: high (dispatchers scan rows). Palette: neutral base,
single accent for primary action, semantic colors for exception severity (with text labels, never
color alone). Type: one sans family, tabular numerals in every numeric column. Radius: small
(`radius-control` on inputs and buttons, `radius-container` on panels only). No gradients. No hero.

Plan, in brief:
- Top bar: search (`/` shortcut), region filter, user menu.
- Left nav: task-based groups (Shipments, Exceptions, Carriers, Reports), `aria-current` on the active item.
- Main: summary strip (counts with thresholds and time range), then the exceptions table with the
  most urgent first.
- Row click opens a side panel; the URL carries the selected shipment so refresh and sharing work.

**Phase 3 — Architecture.**
- Server component page loads the shipment summary and first page of exceptions in parallel
  (`Promise.all`) and streams the table in a `Suspense` boundary with a row-shaped skeleton.
- The filter bar is a client component at the leaf; filters live in search params.
- Side panel: Radix Dialog variant `sheet`, loaded with `next/dynamic` because it is rarely opened.
- Composition: `ShipmentTable` with `Table.Header`, `Table.Row`, `Table.Actions` parts rather than
  boolean props like `showActions`.

**Phase 4 — Implement.**
- Tokens: severity tokens (`--color-severity-high`, etc.) with text labels in each badge.
- Table: sticky header, right-aligned numerics with `tabular-nums`, sort state in the URL, active
  filter chips with remove buttons, row keyboard navigation (arrow keys move focus, Enter opens).
- Empty states: "No exceptions in this region — nice work" for zero, "No results for 'X' — clear
  filters" for filtered zero. Loading: skeleton rows matching final column widths.
- Motion: side panel slides with `ease-out` 240ms and reduced-motion falls back to a fade; no
  animation on the table itself (updates happen many times a minute).
- Error: a failed refresh keeps the last data visible with a banner: "Couldn't refresh. Showing data
  from 10:42. Retry."

**Phase 5 — Verify.**
- Screenshots at 375 (table becomes stacked rows with the key fields), 768, 1440; light and dark.
- Keyboard: Tab order matches visual order; focus returns to the row after the panel closes.
- `browser_evaluate` checks: no horizontal overflow at 375 (the table wraps in its own scroll
  container with a visible affordance, the page does not scroll sideways); body text ≥ 14px; severity
  badges contrast ≥ 4.5:1.
- Console clean. Lint and type check pass.

**Report:** what was verified at which widths; what remains (for example, real-time updates not
tested against a live feed).

## Example 2: Landing page for a developer tool (Persuade mode)

**Brief:** a launch page for a CLI that generates API clients from OpenAPI specs. Audience: backend
developers who skim. One action: install and try.

**Phase 1 — Discover.** Next.js with Tailwind; no existing marketing page. Project has no tokens yet:
create them (`references/design-system.md`).

**Phase 2 — Direction.** Mode: Persuade. Subject: code and specs. The signature element is the
terminal-style demo of generating a client from a real spec, showing the generated file tree. This is
the one memorable moment; everything else stays quiet.

Palette (named): paper `#f7f5f0`, ink `#17151a`, graphite `#5b5762`, signal `#2f6bff`, success
`#1f8a4c`. Type: a grotesque for display and body (one family), a mono for code only (not as decor).
Layout: left-aligned, one column for prose at 68ch, a two-column hero (headline and command left, demo
right) that stacks on mobile with the command first. No gradient text, no eyebrow above the headline,
no three-card feature grid. Features are a short list with real examples, not icon cards.

Copy: the headline names the outcome ("Typed API clients from your OpenAPI spec, in one command").
Primary button: "Install with npm". Secondary link: "Read the docs".

**Phase 3 — Architecture.** Static-first. The hero and demo are server-rendered; the demo animation is
a small client component loaded after the first paint. Code samples use a syntax highlighter loaded
only on the docs route, not the landing page.

**Phase 4 — Implement.**
- Hero: preload the display font, `fetchpriority="high"` only if the hero contains an image (here the
  demo is text, so no image priority is needed).
- Demo: typewriter reveal of the command (one authored moment), then the file tree appears with a
  120ms stagger. Reduced motion shows the final state immediately.
- Copy-to-clipboard button: `<button>`, label changes to "Copied" with `aria-live="polite"`, icon
  hidden from assistive tech.
- Install command: monospace inside a bordered block, `user-select: all`, focus-visible ring.

**Phase 5 — Verify.** Desktop, tablet, and phone screenshots; light theme and a dark theme check.
Confirm LCP element is the headline (text), no layout shift from the font (metric-compatible fallback
set), no horizontal overflow at 320px. Keyboard: the copy button reachable; focus ring visible on the
dark demo background as well. Reduced-motion capture confirmed.

## Example 3: Improving an existing UI (refinement, not redesign)

**Brief:** "The settings page looks rough and feels slow to use. Improve it without changing the
brand."

**Phase 1 — Discover.** Read the existing settings page, its components, and the global CSS. Identify
the incumbent visual identity (type, palette, radius) and keep it. Refinement preserves identity.

**Phase 2 — Diagnose** (the critique checklist, with evidence):
- Hierarchy: every section heading has the same size and weight as its field labels. Fix: section
  heading one step up in size, field labels one step down in weight.
- Spacing: the same `24px` gap between unrelated groups and within groups. Fix: 8px within a group,
  32px between groups.
- Forms: placeholders used as labels on three fields; errors shown only in a toast. Fix: visible labels,
  inline errors, error summary on submit.
- Motion: the save button animates `width` to show a spinner, causing the button to jump. Fix: keep
  width fixed; swap content with an opacity crossfade.
- Accessibility: icon-only "edit" buttons without names; focus ring removed with `outline: none`.
- Performance: the whole page imports a charting library for one sparkline; `transition: all` on
  every input.

**Phase 3 — Plan.** Change in this order: structure and spacing, forms and errors, accessibility,
motion, performance. No new colors or fonts.

**Phase 4 — Implement.** Edit the existing components rather than creating parallel ones. Each change
names its cause in the diff summary. Replace the charting library with the single inline SVG the
sparkline needs (smaller, no dependency).

**Phase 5 — Verify.** Before and after screenshots at 375 and 1440 for the same states (default,
validation error, saving). Keyboard pass through the form. Measured bundle reduction reported with
the analyzer, or a clear statement that it was not measured.

**Phase 6 — Report.** A short table of changes with the reason for each, plus anything deliberately
left unchanged (brand colors, copy the product owner wrote).
