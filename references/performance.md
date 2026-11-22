# Frontend Performance

Load for Core Web Vitals, loading speed, bundle size, rendering, images, fonts, and runtime
responsiveness. Measure before and after every change. Rules here are prioritized by impact.

## Targets (field data at the 75th percentile)

| Metric | Good | What it measures |
|---|---|---|
| LCP | ≤ 2.5s | Largest content appears |
| INP | ≤ 200ms | Input responds to interaction |
| CLS | ≤ 0.1 | Layout does not jump |

Also track TTFB and total JS size on critical routes. Lab tests (Lighthouse, DevTools) guide
changes; field data (CrUX, RUM) judges them.

## Priority order (highest impact first)

1. **Eliminate waterfalls** (critical): run independent async work in parallel; start promises
   early and await late; move `await` into the branch that needs it; stream with Suspense.
2. **Shrink the critical bundle** (critical): dynamic-import heavy components; no barrel imports
   from large libraries; defer analytics and third-party scripts until after hydration; load
   features only when activated.
3. **Server-side work** (high): dedupe per request with `React.cache()`; cache cross-request data
   with an LRU or framework cache; minimize data serialized to client components; use `after()`
   or equivalent for non-blocking work after the response.
4. **Client data fetching** (medium-high): deduplicate with a caching library; version and
   minimize localStorage data; use passive listeners for scroll and touch; register global
   listeners once.
5. **Re-render control** (medium): see `react-architecture.md` re-render discipline.
6. **Rendering** (medium): see below.
7. **JavaScript micro-optimization** (low-medium): only in measured hot loops. Use `Map`/`Set` for
   repeated lookups, early returns, hoisted RegExp, `toSorted()` for immutable sorting, combined
   iterations in hot paths. Do not micro-optimize cold code.

## LCP

- Identify the LCP element (DevTools or `PerformanceObserver`). Make it fast:
  - Preload the hero image or font (`<link rel="preload" as="image">`/`as="font"`), or
    `fetchpriority="high"` on the `<img>`.
  - Do not lazy-load the hero.
  - Serve modern formats (AVIF, WebP) at the displayed size with `srcset` and `sizes`.
  - Avoid rendering the hero behind client-side data fetching; server-render it.
- Remove render-blocking CSS and synchronous scripts from the head where possible.
- Preconnect to critical third-party origins (`<link rel="preconnect">`) only when they are used
  immediately.

## CLS

- Set `width` and `height` (or `aspect-ratio`) on every image, video and iframe.
- Reserve space for ads, embeds, banners and late-loaded content.
- Use `font-display: swap` with a metric-compatible fallback (`size-adjust`,
  `ascent-override`, `descent-override`) so text swap does not reflow.
- Do not insert content above existing content after load (banners, consent bars) unless they
  are fixed or overlay.
- Animate `transform` instead of changing layout properties.

## INP

- Keep event handlers short. Move non-urgent work out of the input path with `startTransition`,
  `useDeferredValue`, or `requestIdleCallback` for non-critical work.
- Avoid heavy synchronous work during input; split long tasks into chunks yielding to the main
  thread.
- Debounce remote search; throttle high-frequency handlers that do real work.
- Batch DOM reads and writes; avoid interleaving layout reads (`offsetHeight`,
  `getBoundingClientRect`, `scrollTop`) with writes in loops.
- Group CSS changes through classes or `cssText` instead of many single-property writes.
- Keep controlled inputs cheap per keystroke; for expensive filtering, use uncontrolled inputs
  or defer the derived list.

## Images

- Explicit dimensions on every `<img>`.
- `loading="lazy"` for below-the-fold images; `fetchpriority="high"` for the hero only.
- Modern formats and responsive sizes. Compress; do not ship full-resolution assets into thumbnails.
- Prefer `<video autoplay muted loop playsinline>` over animated GIFs, with a still fallback and a
  `prefers-reduced-motion` check.
- Decorative images: `alt=""`. Meaningful images: descriptive `alt`.
- Large lists of images: `content-visibility: auto` or virtualization.

## Fonts

- Self-host or use the framework's font loader; subset to used characters/scripts.
- Load only the weights and styles in use; prefer variable fonts when several weights are used.
- `font-display: swap` (or `optional` for non-critical text) and preload the one critical font
  file.
- Match fallback metrics to avoid layout shift.

## Long lists and rendering

- Lists over ~50 items: virtualize (e.g. `@tanstack/react-virtual`, `virtua`) or paginate.
- `content-visibility: auto` with `contain-intrinsic-size` for long off-screen sections.
- Keep static JSX out of render paths (hoist constants).
- Use `Activity` or CSS hiding to keep state for hidden panels without rerendering.
- Use the native `<dialog>`, `<details>`, and `popover` where they fit, instead of heavy JS widgets.

## Scripts and third parties

- `defer` or `async` on non-critical script tags.
- Load analytics and chat after hydration or on user intent.
- Audit third-party scripts quarterly; remove what no one uses.
- Measure the cost of each third party (main-thread time) before adding one.

## Motion performance

See `motion.md`. Summary: animate `transform` and `opacity`; avoid per-frame React state updates;
avoid layout-driving properties; bound blur and shadow work; `will-change` only after observed need.

## Measuring

- Lighthouse and PageSpeed Insights for lab scores; CrUX or RUM for field data.
- Chrome DevTools Performance panel: long tasks, layout thrash, dropped frames.
- Bundle analysis (`@next/bundle-analyzer`, `rollup-plugin-visualizer`, or the framework's analyzer).
- Test on a mid-range Android device or CPU throttling (4×) to represent real users. A fast laptop
  hides most problems.
- Record before and after numbers in the report.

## Verify (performance)

- LCP element identified and prioritized.
- No images without dimensions; no fonts causing shift.
- No layout-driving animation; no per-frame state updates.
- Critical bundle size reviewed; large dependencies justified.
- Before/after numbers reported, or a clear statement that they were not measured.
