# Accessibility

Load for any interface work. Target WCAG 2.2 Level AA. Accessibility is part of the design, not a
final pass: fix structure while building.

## Semantic HTML first

- Use the element that means what it does: `<button>` for actions, `<a href>` for navigation,
  `<label>` for form labels, `<table>` for tabular data, `<nav>`, `<main>`, `<header>`,
  `<footer>`, `<section>` with a heading, `<dialog>` for modal dialogs, `<details>`/`<summary>` for
  disclosure.
- Never `<div onClick>`. If you truly cannot use a native control, use `role` + `tabIndex=0` +
  full keyboard handling, and prefer finding the native element first.
- Headings are hierarchical: one `h1` per page, no skipped levels, headings describe sections.
- Include a skip link to main content as the first focusable element.

## Keyboard

- Every interactive element is reachable with Tab and operable with Enter/Space as appropriate.
- Logical focus order matches visual order. No positive `tabIndex`.
- Visible focus indicator on every focusable element: `:focus-visible` with a 2px+ ring that has
  ≥ 3:1 contrast against adjacent colors. Never `outline: none` without a replacement.
- Focus is not obscured by sticky headers or overlays (WCAG 2.4.11).
- Dialogs: move focus into the dialog on open, trap focus inside while open, return focus to the
  trigger on close, close on Escape.
- Menus, tabs, comboboxes and listboxes follow WAI-ARIA authoring practices for arrow-key
  behavior. Use a headless primitive (Radix, React Aria, or a native control) rather than
  reinventing this.
- Keyboard shortcuts must not conflict with assistive tech or browser shortcuts, and must be
  discoverable or remappable.

## Names, roles and states

- Every control has an accessible name: a visible label, `aria-label`, or `aria-labelledby`.
- Icon-only buttons need `aria-label` (or visually hidden text).
- Decorative icons beside text: `aria-hidden="true"`.
- Expose state: `aria-expanded`, `aria-pressed`, `aria-selected`, `aria-current`, `aria-invalid`,
  `aria-disabled` only when the native `disabled` attribute cannot be used.
- Prefer native `disabled` to `aria-disabled` for buttons and inputs.
- ARIA is a last resort: no ARIA is better than wrong ARIA. Do not put `role` on an element that
  already has the right semantics.
- Async status (toasts, validation, loading results): `role="status"` (polite) or `role="alert"`
  (assertive, only for errors that need immediate attention). Keep the live region present in the
  DOM before content changes.
- Use `aria-busy` on regions that are loading.

## Forms

- Every input has a programmatic label. Placeholder is not a label.
- Errors: text next to the field, linked with `aria-describedby`, `aria-invalid="true"` on the
  field. On submit with several errors, also show an error summary that links to each field and
  receives focus.
- Use correct `type`, `autocomplete` and `inputmode` so screen readers, password managers and
  keyboards behave correctly.
- Do not require re-entering information in the same process. Keep typed data on error.
- Authentication: allow paste and password managers; do not rely on cognitive tests (WCAG 3.3.8).
- Required fields marked in text ("required" or an asterisk with a legend), not by color alone.

## Color and contrast

- Body text ≥ 4.5:1; large text (≥ 24px, or ≥ 18.66px bold) ≥ 3:1; UI component boundaries and
  meaningful icons and graphics ≥ 3:1 against adjacent colors.
- Placeholder and disabled text are exempt from contrast minimums, but still must be readable
  enough to be useful; do not use them for required information.
- Color is never the only signal: pair with text, icon, shape or pattern (charts, status badges,
  form errors).
- Check both themes independently. Dark mode is not an inversion.
- Set `color-scheme: light dark` (or `dark` on `<html>` for dark themes) so native controls and
  scrollbars match.

## Touch and pointer

- Target size ≥ 24×24 CSS px (WCAG 2.2 AA, with spacing); aim for 44×44 for primary controls and
  mobile. Expand the hit area with padding or a pseudo-element when the visual is smaller.
- Single-pointer alternatives for every drag, multi-touch, or path-based gesture (WCAG 2.5.1, 2.5.7).
- Do not trigger actions on pointer down for destructive or irreversible actions; use click/up.
- Avoid hover-only disclosure; hover content must also appear on focus and be dismissible
  (WCAG 1.4.13): dismissible without moving the pointer, persistent until dismissed.

## Motion and media

- `prefers-reduced-motion`: remove transform-based movement, parallax, autoplay and looping
  motion; keep meaningful opacity and color feedback. See `motion.md`.
- Auto-playing content longer than 5 seconds needs pause, stop or hide controls.
- Flashing: nothing flashes more than three times per second.
- Video: captions, and audio descriptions when meaning depends on visuals. Audio: transcripts.
- Media controls are keyboard-operable and labeled.

## Text and zoom

- Text resizes to 200% without loss of content or function. Test at 200% zoom and at 400% reflow
  (320px wide content without horizontal scroll, WCAG 1.4.10).
- Do not disable zoom: no `user-scalable=no` or `maximum-scale=1`.
- Use relative units (rem, em) for text; respect the user's font size setting.
- Line height ≥ 1.5 for body; paragraph spacing ≥ 2× font size is recommended by WCAG 1.4.12
  text-spacing requirements. Do not clip content when spacing is increased.
- Language: `<html lang="…">`; `lang` on inline passages in another language.

## Tables, lists and data

- Tables: `<th scope="col|row">`, a caption or accessible name, no layout tables.
- Charts: a text summary and a data table alternative; color is not the only encoding.
- Lists: use `<ul>`/`<ol>` for real lists, including navigation lists.

## Content and copy

- Link text makes sense out of context: avoid "click here" and repeated "Learn more" without
  context (use "Learn more about billing").
- Page titles describe the page and product. Update `document.title` on route changes.
- Error messages say what happened and how to fix it.
- Dates, numbers and currencies via `Intl` for locale-correct reading.
- `translate="no"` for brand names and code identifiers that should not be auto-translated.

## Screen reader checks

- Read the page's structure with a rotor or heading list (VoiceOver or NVDA): headings, landmarks,
  link list, form controls.
- Verify announcements for: save success, validation errors, loading completion, new items.
- Verify focus after: opening and closing dialogs, deleting an item, route change, form error.

## Tools

- axe DevTools or `@axe-core/playwright` for automated checks (catches roughly a third to half of
  issues; manual review is still required).
- Lighthouse accessibility audit as a lab signal.
- Keyboard-only walkthrough of every flow.
- Zoom to 200% and 400%; Windows high-contrast or forced-colors mode for custom controls.

## Verify (accessibility)

- Keyboard-only walkthrough completes every task.
- Every control has a name; every icon-only control has a label.
- Focus indicator visible on every focusable element; focus moves logically for dialogs and errors.
- Contrast values checked (computed, not guessed) in both themes.
- Reduced-motion path reviewed.
- Automated scan run and findings either fixed or justified.
