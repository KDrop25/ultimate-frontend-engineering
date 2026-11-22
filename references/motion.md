# Motion and Interaction Design

Load for any animation, transition, micro-interaction, gesture, page transition, skeleton, or
scroll effect. Motion must explain state, relationship, feedback or hierarchy. Decoration
that repeats often is debt.

## Decide whether to animate

| How often the user sees it | Decision |
|---|---|
| 100+ times a day (keyboard shortcuts, command palette toggle) | No animation |
| Tens of times a day (hover, list navigation, toggles) | Remove or reduce to ~100ms |
| Occasionally (menus, drawers, toasts, dialogs) | Standard animation |
| Rarely (onboarding, celebrations, first-run) | Room for delight |

Valid purposes: acknowledge an action, show a state change, preserve spatial continuity,
prevent a jarring appearance, direct attention once. "It looks cool" on frequent UI is not
a purpose.

## Choose the motion thesis

For each surface, write:
- **Focal moment:** the one sequence worth authoring (may be none on Operate surfaces).
- **Continuity:** what state or layout change needs explaining.
- **Feedback:** which controls must acknowledge input.
- **Budget:** what runs during heavy work (navigation, fetching, hydration) and must stay smooth.

Generic fade-and-rise on every section is not a thesis. Use one orchestrated page-load moment
or one reveal rather than scattered effects.

## Easing and duration

Default curves (use these or a similar strong custom curve from easing.dev / easings.co):

```css
--ease-out: cubic-bezier(0.23, 1, 0.32, 1);      /* entering, responding */
--ease-in-out: cubic-bezier(0.77, 0, 0.175, 1);  /* moving on screen */
--ease-drawer: cubic-bezier(0.32, 0.72, 0, 1);   /* sheets and drawers */
--duration-press: 120ms;
--duration-small: 180ms;   /* tooltips, toggles */
--duration-base: 240ms;    /* dropdowns, selects */
--duration-large: 380ms;   /* modals, drawers (max ~500ms) */
```

- Entering or exiting → `ease-out`. Moving or morphing → `ease-in-out`. Hover or color → `ease`.
  Constant motion (progress, marquee) → `linear`.
- Never `ease-in` on UI: it delays the moment the user is watching.
- Exit faster than enter (for example enter 250ms, exit 180ms).
- Keep UI under 300ms except modals, drawers, sheets and toasts.
- Feedback that feels slow reads as latency; a faster spinner makes load feel faster.

## Physicality

- Buttons: `transform: scale(0.97)` on `:active` with `transition: transform 160ms var(--ease-out)`.
  Range 0.95–0.98.
- Never animate from `scale(0)`. Start at 0.9–0.97 with `opacity: 0`.
- Popovers, menus and tooltips scale from their trigger: `transform-origin: var(--transform-origin)`.
  Modals stay centered (`transform-origin: center`).
- Translate with percentages (`translateY(100%)`) for elements of unknown size.
- `scale()` also scales children; use it for press feedback.

## Transitions over keyframes for interactive UI

CSS transitions can be interrupted and retargeted; keyframes restart from zero. Use
transitions for anything users can trigger repeatedly (toasts, toggles, tabs, accordions).

```css
.toast {
  transition: transform 400ms ease, opacity 400ms ease;
  @starting-style { opacity: 0; transform: translateY(100%); }
}
```

Fallback without `@starting-style`: set `data-mounted` after first paint.

## Springs (gesture and momentum)

Use springs for drag with momentum, interruptible gestures and decorative pointer-follow.

```js
{ type: "spring", duration: 0.5, bounce: 0.2 }   // readable form (Motion)
{ type: "spring", stiffness: 300, damping: 30 }  // physics form
```

Keep bounce 0.1–0.3 and reserve it for drag-to-dismiss and playful surfaces. Springs keep
velocity when interrupted; keyframes do not.

Pointer-follow effects: interpolate with a spring (`useSpring`), not the raw pointer value,
and only when decorative.

## Gestures

- Momentum dismissal: dismiss when `distance / elapsedMs > ~0.11` even under the threshold.
- Damping past boundaries: movement shrinks as the user drags further (friction, not a wall).
- Pointer capture on drag start; ignore extra touch points during a drag.
- Every drag, swipe or pinch needs a button or keyboard alternative.

## clip-path and masking

- `clip-path: inset(...)` reveals and tab-indicator color transitions (duplicate the tab row,
  clip the active copy, animate the clip).
- Reveal on scroll: `inset(0 0 100% 0)` → `inset(0 0 0 0)` with IntersectionObserver, once.
- Masking a messy crossfade: a 2px `filter: blur()` during the transition.
- Keep blur under 20px; heavy blur is expensive, especially in Safari.

## Stagger

- Stagger lists entering together: 30–80ms between items, capped total delay (~300ms).
- Never block interaction while a stagger plays.
- Do not stagger every scrolled section; one authored entrance per container at most.

## Performance rules for motion

- Animate `transform` and `opacity` by default. `clip-path`, `filter` (small blur), and
  `background-color` are acceptable when they stay smooth.
- Animating `width`, `height`, `top`, `left`, `margin` or `padding` triggers layout each frame.
  Replace with `transform` (or FLIP) when the visual result allows.
- Shadow animation: prefer `filter: drop-shadow()` over `box-shadow` for repeated effects.
- CSS variables inherit: setting `--x` on a parent restyles every child each frame. Set
  `transform` directly on the moving element.
- Do not drive per-frame animation through React state (`setState` in `pointermove`,
  `scroll`, or `requestAnimationFrame`). Write to `ref.current.style` or a motion value.
- Motion `x`/`y`/`scale` shorthands are hardware-accelerated from `motion` 14.2; on older
  versions use the full `transform` string, or CSS/WAAPI for animations that must survive a
  busy main thread.
- Layout animations (`layout`, `layoutId`) and `onUpdate` run on the main thread; avoid them
  during page transitions or data loading.
- `will-change` only on the element that animates, only after you observe a shift or measured
  dropped frames. Never on `*` or large containers.
- Animation runtime choice:
  - CSS transitions/keyframes: predetermined and state-driven.
  - Web Animations API: programmatic, interruptible, no library.
  - Motion (or the project's existing motion library): springs, gestures, layout animation,
    exit animations for unmounted components.
  - Do not add a library for an effect the existing stack expresses cleanly.

## Hover

- Gate hover effects: `@media (hover: hover) and (pointer: fine) { … }`.
- Hover must not move a neighbor or change the pointer target. Animate a child, not the
  hovered element, to avoid edge flicker.
- Delay hover previews with `transition-delay` on the hover state only, so leaving stays instant.

## Loading animation and skeletons

- Skeletons match the final layout's shape and size; use them for content, not spinners.
- Spinners only for short, indeterminate actions inside a button or small region.
- Shimmer is optional; if used, animate `background-position` or a transform pseudo-element
  and stop it under reduced motion.

## Page transitions

- Keep route transitions short (≤ 250ms) and avoid blocking input.
- Use View Transitions or shared-element motion when continuity across states is the point.
- Never hold content hostage to an entrance animation.

## Scroll-driven motion

- Only when the scroll relationship carries meaning (progress, pinned storytelling).
- Provide a static fallback; content must be visible before scripts run.
- Use the native scroll-driven CSS timeline where supported, with a JS fallback if needed.

## Accessibility (reduced motion)

```css
@media (prefers-reduced-motion: reduce) {
  .element { transition-duration: 1ms; animation-duration: 1ms; }
}
```

Better: keep opacity and color transitions that carry meaning, remove transform-based movement,
parallax, auto-play, and looping motion. Reduced motion means fewer and gentler effects, not zero.

Autoplay motion longer than 5 seconds next to other content needs pause, stop or hide controls.
Muted decorative loops stop under reduced motion and when offscreen.

## Debugging motion

- Slow motion: multiply durations by 2–5× temporarily; check that colors crossfade cleanly, that
  coordinated properties stay in sync, and that `transform-origin` is right.
- DevTools Animations panel for timing; Performance panel for dropped frames.
- Test gestures on a real device over remote debugging, not only the desktop emulator.
- Review animations after a break; imperfections surface with fresh eyes.

## Motion review table (use for critique)

| Before | After | Why |
|---|---|---|
| `transition: all 300ms` | `transition: transform 200ms var(--ease-out)` | explicit properties; `all` animates accidents |
| `scale(0)` entrance | `scale(0.95)` + `opacity: 0` | nothing appears from nothing |
| `ease-in` dropdown | `ease-out` custom curve | faster initial response |
| No `:active` on button | `scale(0.97)` on `:active` | press feedback |
| Keyboard action animated | remove | repeated too often to animate |
| Modal with `transform-origin` at trigger | keep centered | modals are not trigger-anchored |
| `margin-left` sidebar animation | `translateX` | no layout on every frame |
