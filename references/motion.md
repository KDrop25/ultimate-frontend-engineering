# Motion and Interaction Design

Load for any animation, transition, micro-interaction, gesture, page transition, skeleton, or
scroll effect. Motion must explain state, relationship, feedback or hierarchy. Decoration
that repeats often is debt.

## Decide whether to animate

| How often the user sees it | Decision |
|---|---|
| Multiple times per hour, every day (keyboard shortcuts, command palette) | No animation |
| Many times per session (hover states, list navigation, toggles) | Remove, or keep under ~100ms |
| A few times per session (menus, drawers, toasts, dialogs) | Standard, restrained animation |
| Rarely seen (onboarding, celebrations, first-run states) | Room for expressive motion |

Motion is justified when it acknowledges an action, shows a state change, keeps spatial
context, avoids an abrupt appearance, or draws attention once. Decoration on frequently used
controls costs more attention than it earns.

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
--ease-out: cubic-bezier(0.33, 1, 0.68, 1);      /* entering, responding */
--ease-in-out: cubic-bezier(0.65, 0, 0.35, 1);   /* moving on screen */
--duration-press: 120ms;
--duration-small: 180ms;   /* tooltips, toggles */
--duration-base: 240ms;    /* dropdowns, selects */
--duration-large: 380ms;   /* modals, drawers (max ~500ms) */
```

- Pick the curve by the motion's direction: an element arriving or responding uses an
  ease-out curve; an element travelling across the screen uses ease-in-out; color and
  opacity changes use a plain ease; continuous progress indicators use linear.
- Avoid ease-in for interface responses. It keeps the element still at the start, which is when
  the user is watching most closely.
- Let exits run shorter than entrances (for example 250ms in, 180ms out).
- Keep most interface motion under 300ms. Modals, drawers, sheets, and toasts may run a little longer.
- Perceived speed matters: a faster spinner makes loading feel quicker.

## Physicality

- Press feedback: scale a button to about 0.97 on `:active`, with a short transform transition
  (roughly 160ms). Keep the scale between 0.95 and 0.98.
- Do not start an entrance at `scale(0)`. Start near 0.9–0.97 combined with `opacity: 0`.
- Anchor popovers, menus, and tooltips to their trigger with `transform-origin: var(--transform-origin)`
  (a variable your positioning library provides). Modals stay centered (`transform-origin: center`).
- Use percentage translations (`translateY(100%)`) for elements whose size you do not know.
- `scale()` also scales child content, which suits press feedback.

## Transitions over keyframes for interactive UI

When a state changes mid-transition, a CSS transition retargets from its current position.
A keyframe animation restarts from its beginning. Prefer transitions for controls a user can
trigger repeatedly, such as toasts, toggles, tabs, and accordions.

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
{ type: "spring", duration: 0.5, bounce: 0.2 }   // duration + bounce form
{ type: "spring", stiffness: 300, damping: 30 }  // stiffness + damping form
```

Keep bounce low (roughly 0.1–0.3) and limit it to drag-to-dismiss or playful surfaces. A spring
carries its current velocity into a new target when interrupted; a keyframe animation does not.

Pointer-follow effects: interpolate with a spring (`useSpring`), not the raw pointer value,
and only when decorative.

## Gestures

- Momentum dismissal: dismiss when `distance / elapsedMs > ~0.11` even under the threshold.
- Damping past boundaries: movement shrinks as the user drags further (friction, not a wall).
- Pointer capture on drag start; ignore extra touch points during a drag.
- Every drag, swipe or pinch needs a button or keyboard alternative.

## clip-path and masking

- A `clip-path: inset(...)` can reveal content or move a highlight between items. For a
  two-color tab switch, render a second copy of the tab row in the active colors and reveal
  it with an animated clip.
- For a reveal on scroll, start from a fully clipped inset and animate to none once the
  element enters the viewport (IntersectionObserver, one time only).
- A brief 2px `filter: blur()` can soften a crossfade that shows two states at once.
- Keep blur small (under about 20px). Large blurs are costly, particularly in Safari.

## Stagger

- Stagger lists entering together: 30–80ms between items, capped total delay (~300ms).
- Never block interaction while a stagger plays.
- Do not stagger every scrolled section; one authored entrance per container at most.

## Performance rules for motion

- Prefer animating `transform` and `opacity`. `clip-path`, a small `filter` blur, and
  `background-color` are workable when they remain smooth on target devices.
- Changing `width`, `height`, `top`, `left`, `margin`, or `padding` forces layout on every frame.
  Use `transform`, or a FLIP technique, when it produces the same visual result.
- For animated shadows, `filter: drop-shadow()` is usually cheaper than animating `box-shadow`.
- A CSS custom property set on a parent forces style recalculation for all of its children on
  each change. Write `transform` directly to the element that moves.
- Keep per-frame values out of React state. Updating state inside `pointermove`, scroll, or
  `requestAnimationFrame` handlers re-renders the tree every frame. Write to a ref's style or
  a motion value instead.
- In the `motion` library, the `x`/`y`/`scale` shorthands run on the compositor from version
  14.2. Older versions run them on the main thread, so use the full `transform` string there,
  or CSS/WAAPI for anything that must keep running while the main thread is busy.
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
- A hover effect should not shift a neighboring element or the hit area under the pointer.
  Animate a child inside the hovered element so the pointer target stays put.
- For hover previews, add `transition-delay` only to the entering state, so leaving still feels immediate.

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

Content that moves on its own for more than five seconds needs a way to pause, stop, or hide it
(WCAG 2.2.2). Decorative loops also stop when reduced motion is requested or when they leave the viewport.

## Debugging motion

- Slow motion: multiply durations by 2–5× temporarily; check that colors crossfade cleanly, that
  coordinated properties stay in sync, and that `transform-origin` is right.
- DevTools Animations panel for timing; Performance panel for dropped frames.
- Test gestures on a real device over remote debugging, not only the desktop emulator.
- Review animations after a break; imperfections surface with fresh eyes.

## Motion review table (use for critique)

| Before | After | Why |
|---|---|---|
| `transition: all 300ms` | `transition: transform 200ms var(--ease-out)` | name the properties; `all` also animates unplanned changes |
| Entrance from `scale(0)` | Start at `scale(0.95)` with `opacity: 0` | a visible start state reads as natural |
| `ease-in` on a dropdown | An ease-out curve | the early frames respond sooner |
| Button without a pressed state | `scale(0.97)` on `:active` | confirms the press |
| Keyboard shortcut triggers an animation | Remove it | too frequent to justify motion |
| Modal that slides out of its trigger button | Keep it centered in the viewport | a dialog is not anchored to one element |
| `margin-left` sidebar animation | `translateX` | no layout on every frame |
