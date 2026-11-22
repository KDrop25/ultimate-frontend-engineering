# Visual QA

Load before finishing any frontend task, and whenever reviewing existing UI. A build passing is not
evidence the interface works. Never claim visual testing you did not perform: if a browser was not
available, say so and list what was verified by reading code.

## When browser tools are available

Available tools include the Playwright MCP (`browser_navigate`, `browser_resize`,
`browser_take_screenshot`, `browser_snapshot`, `browser_evaluate`, `browser_console_messages`,
`browser_press_key`) and the Claude-in-Chrome tools. Load them with ToolSearch first, in one call.

### The loop

1. **Start the app** with the project's dev command (check `package.json` scripts). Wait for it to
   respond. If it fails, fix the startup error first; it is in scope.
2. **Open each relevant page** and wait for network idle and fonts to load.
3. **Capture at three widths:** 375px (phone), 768px (tablet), 1440px (desktop). Also check 320px
   for reflow issues on the main page.
4. **Capture both themes** (light and dark) when the product supports both.
5. **Capture reduced motion** (`browser_emulate_media` / `prefers-reduced-motion: reduce`) for pages
   with motion.
6. **Test every interaction:** click, type, open/close menus and dialogs, submit forms with invalid
   and valid data, trigger each empty, loading and error state if reachable.
7. **Keyboard pass:** Tab through the whole page; confirm visible focus, logical order, no traps,
   Escape closes overlays, focus returns to the trigger.
8. **Measure** with `browser_evaluate` for things eyes miss: `document.documentElement.scrollWidth >
   window.innerWidth` (horizontal overflow); computed font sizes on body text; contrast of key text
   pairs; images missing `alt` or dimensions; elements without accessible names.
9. **Console:** read errors and warnings. Ignore nothing without a reason.
10. **Critique** the screenshots against the checklist below. Score each 1–5; fix anything under 4.
11. **Fix** the top issues, re-render, and check again. Do at least two loops on new work.
12. **Report** what was checked, at which widths, and what remains.

### Efficient capture

- Batch the checks for one viewport into one pass before taking another screenshot.
- Use `browser_snapshot` (accessibility tree) to check names, roles and structure without images.
- Keep screenshots of the actual changed regions; full-page captures only where the layout is the
  question.

## When no browser is available

State it plainly. Then verify by reading: check responsive classes at each breakpoint, focus
styles, `alt` and labels, token usage, state coverage, and reduced-motion rules. Report these as
code-level checks, not visual ones.

## Critique checklist (score 1–5)

1. **Hierarchy:** in three seconds, is it clear what this is and what to do next?
2. **Typography:** clear scale steps; body ≥16px; headings balanced; measure 45–75ch for prose; no
   widows in headings; numerals aligned in tables.
3. **Rhythm:** section spacing consistent and from tokens; related items closer than unrelated.
4. **Alignment:** edges line up across sections; nothing centered by accident; grid respected.
5. **Color and contrast:** one accent used for action; text ≥ 4.5:1; dark theme designed, not
   inverted; state never by color alone.
6. **Originality:** would this be a competitor's site if the logo were swapped? If yes, strengthen
   the signature element.
7. **Details:** focus rings, hover states, favicon, title, empty and error states, form messages.
8. **Mobile:** tap targets ≥ 44px; no hover-only affordance; nav works; hero fits; no horizontal
   scroll; text not shrunk below 16px.
9. **Performance:** hero image prioritized and sized; fonts preloaded and subset; no layout shift
   from late content.
10. **Motion:** the focal motion is intentional; no scattered effects; reduced-motion path works;
    no layout-driving animation.

## Pass / fail gates

Must pass before delivery:
- No horizontal overflow at 320, 375, 768, 1440.
- No console errors. Warnings reviewed.
- Every interactive element reachable and operable by keyboard; focus visible.
- Every image has `alt` (or `alt=""` if decorative) and dimensions.
- Every control has an accessible name.
- Body text contrast ≥ 4.5:1 in every theme checked.
- Reduced-motion path reviewed for every animated surface.
- Every state from the UX states table exists or is explicitly out of scope.
- Lint, type check and relevant tests pass.

## Automated accessibility scan

- `@axe-core/playwright` in an existing Playwright setup, or the axe DevTools extension.
- Treat automated results as a floor; manual keyboard and screen-reader checks still apply.

## Reviewing existing UI (audit mode)

Use the same checklist plus this code-level scan on the files in scope. Report findings as
`file:line` with the issue and a fix:

- Icon-only buttons missing `aria-label`
- Inputs without labels; placeholder used as the only label
- `<div>` or `<span>` with click handlers (should be `<button>`/`<a>`)
- `outline: none` without a focus replacement
- `transition: all`
- Animations without a `prefers-reduced-motion` path
- Images without `width`/`height` or `alt`
- Hardcoded date and number formats (use `Intl`)
- `user-scalable=no` or `maximum-scale=1`
- Large unvirtualized lists
- `autoFocus` without a clear reason
- Raw hex values or magic spacing values in components
- Hover-only interactions
- Client components at the top of the tree that could be server components

Format: one line per finding (`file:line — problem — fix`), grouped by file, then a short summary.
