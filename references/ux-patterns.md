# UX Patterns

Load for information architecture, navigation, forms, tables, search and filters, dashboards,
empty/loading/error states, onboarding, and copy. Goal: intuitive interfaces, not merely
attractive ones.

## Start from the user journey

1. Who is the user and what job brings them here?
2. What is the first thing they must see to decide what to do?
3. What is the primary action? Make it obvious; make everything else subordinate.
4. What happens on failure, on empty data, while loading, and after success?

## Information architecture and navigation

- Group navigation by user tasks, not by database tables or org charts.
- Max ~7 top-level items on desktop; on mobile, bottom navigation ≤ 5 items.
- Make the current location visible (active state plus `aria-current="page"`).
- Predictable back behavior: navigation state lives in the URL so back, refresh and
  deep links work. Filters, tabs, pagination and open panels belong in query params when
  they should be shareable.
- Responsive navigation: collapse to a menu button on narrow widths, but keep the most
  frequent destinations reachable in one tap. Sheets and drawers need a visible close control.
- Breadcrumbs for hierarchies deeper than two levels.

## Interaction patterns

- Use the native control first: `<button>`, `<a>`, `<input>`, `<select>`, `<dialog>`.
- Menus, comboboxes, tabs, tooltips and dialogs follow WAI-ARIA authoring practice; Radix or
  a similar headless primitive provides this correctly (see `component-ecosystem.md`).
- Confirm or offer undo for destructive actions. Undo is usually better than confirmation for
  reversible actions.
- Optimistic updates for low-risk actions (toggling a favorite), with rollback and an
  explicit error message on failure.
- Tap targets ≥ 44×44 CSS px (24×24 minimum per WCAG 2.2 AA with spacing); expand the hit
  area when the visual mark is small.
- Hover must never be the only way to reveal something. Touch has no hover.

## Forms

- Visible label for every input. Placeholder text is an example, not a label; it ends with
  `…` when it shows a pattern (for example `name@company.com…`).
- Correct `type`, `inputmode`, `autocomplete` and `name` so password managers and mobile
  keyboards work. `autocomplete="off"` only for non-auth fields that should not be autofilled.
- Validate on blur or after the first submit, not on every keystroke for new fields.
- Errors appear next to the field, in text, with the fix ("Use at least 8 characters").
  After a failed submit, move focus to the first error or to an error summary linked to fields.
- Keep the submit button enabled until the request starts; show progress during the request.
- Never block paste. Keep typed values on failure. Warn before navigating away from unsaved
  changes.
- Group related fields with `<fieldset>` and `<legend>`. Put optional fields in progressive
  disclosure only when they are truly secondary.
- Checkboxes and radios: the label and control share one hit target.

## Data tables and lists

- Tables for comparable records; use `<table>` with `<th scope>` headers. Lists for
  sequences of similar items that are not compared column by column.
- Right-align numeric columns; use tabular numerals; keep units in headers.
- Sticky header on long tables; keep the first column visible when horizontal scrolling is
  unavoidable.
- Sort, filter and search state in the URL. Show the active filters as removable chips.
- Row actions: a primary action on click of the row or a visible action button; avoid icon-only
  actions without labels.
- Large lists (>50 rows): paginate or virtualize. Show counts ("1–25 of 340").
- On mobile, convert each row to a stacked card that keeps the key fields and identifies the
  record uniquely.

## Search and filtering

- Search is the fastest path for expert users: keep it visible, support keyboard shortcut
  (`/` or `Cmd/Ctrl+K`) when the product has many objects.
- Debounce remote search (~200–300ms); show results count and an empty state with a clear
  next step.
- Filters: most common first, applied immediately with a visible result count. Apply/cancel
  buttons only for expensive or multi-step filters.

## Dashboards (Operate mode)

- Put the answer to the user's most frequent question top-left. Summary numbers first, then
  the detail that explains them.
- Each metric needs: a label, the value, the time range, and a comparison or threshold when
  it helps decisions.
- Chart choice by question: trend → line; composition → stacked bars or a single share bar
  (pie only for two or three parts); comparison → bars; distribution → histogram. Always label
  axes and provide a text or table alternative.
- Color in charts must not carry meaning alone; pair with labels or patterns.
- Density: operators want density (8–32px spacing scale); executives want space. Pick from the
  real use scene.
- Avoid decorative KPI tiles that repeat the same number in three places.

## States (every screen needs all that apply)

| State | Pattern |
|---|---|
| Loading (content) | Skeleton matching the final layout's shape; announce with `aria-busy` |
| Loading (action) | Button shows progress and stays the same width; text changes to "Saving…" |
| Empty (first use) | Explain the value, one primary action to create the first item |
| Empty (filtered) | "No results for X" plus a control to clear filters |
| Error (field) | Inline, next to the field, with the fix |
| Error (page or section) | What failed, what the user can do (retry, contact, return), what is preserved |
| Success | Confirm in place with a short message; toasts for non-blocking results only |
| Permission | Explain who can grant access and how to request it |
| Offline | Say what still works and what will sync |
| Partial | Show available data and mark what is missing |

## Empty states and onboarding

- An empty screen is an invitation, not a dead end: name the next step.
- Onboarding: ask the minimum; defer everything else to when it becomes relevant.
- Show the product's value with the user's own data as early as possible (sample data
  clearly labeled as such).
- Checklists for activation are fine when they track real milestones; remove them when done.

## Copy

- Use the product's own vocabulary consistently. A button that says "Publish" produces a
  toast that says "Published".
- Buttons name the action: "Save changes", "Invite teammate", not "Submit" or "OK".
- Sentence case for UI text; Title Case only where the product already uses it. Second person.
- Errors: no apology filler, name the problem, give the fix.
- Use real numbers and realistic content when designing. Lorem ipsum hides overflow.
- Use `…` for ellipsis and curly quotes in display copy. Use `Intl.DateTimeFormat` and
  `Intl.NumberFormat` for dates and numbers.

## Mobile-first interaction

- Design the smallest viewport first (320–375px). Recompose the layout for mobile rather than
  shrinking desktop.
- Single primary column; secondary content below or in a sheet.
- Thumb-zone: place frequent actions low on screen or in a bottom bar.
- Respect safe areas (`env(safe-area-inset-*)`) for fixed headers and bottom bars.
- Avoid horizontal scroll except for deliberate carousels and tables with a clear affordance.
- Avoid `user-scalable=no` or `maximum-scale=1`; zoom must work.

## Verify (UX)

- Can a first-time user find the primary action within a few seconds?
- Does every error state tell the user what to do next?
- Does every empty, loading and error state exist (checked against the table above)?
- Does keyboard-only navigation reach every action in a sensible order?
