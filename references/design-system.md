# Design System

Load when defining tokens, theming, component variants, or extending an existing design system.
Goal: every page and component draws from one source of truth.

## First: does a system already exist?

Search for: CSS custom properties on `:root`, a theme file, `tailwind.config.*` or the Tailwind v4
`@theme` block, `tokens/` or `styles/` directories, a component library folder. If one exists,
extend it. Do not create a parallel system.

## Token layers

Use three layers. Components reference semantic tokens only.

1. **Primitive (raw) tokens:** the palette and scales. `--blue-600: #…`, `--space-4: 1rem`.
2. **Semantic tokens:** purpose. `--color-text-primary`, `--color-surface-raised`,
   `--color-border-subtle`, `--color-accent`, `--color-focus-ring`, `--radius-control`,
   `--shadow-overlay`, `--duration-base`, `--ease-out`.
3. **Component tokens (optional):** only when a component needs its own knob, for example
   `--button-padding-x`. Skip until a second use proves the need.

Name tokens by role, not value (`--color-danger`, not `--color-red`).

## Token set

```css
:root {
  /* Color: semantic, light theme */
  --color-bg: #fafaf9;
  --color-surface: #ffffff;
  --color-surface-raised: #ffffff;
  --color-text: #1c1917;
  --color-text-muted: #57534e;          /* ≥4.5:1 on --color-surface */
  --color-border: #e7e5e4;
  --color-accent: #2f5bea;
  --color-accent-text: #ffffff;
  --color-focus-ring: #2f5bea;
  --color-success: #15803d;
  --color-warning: #b45309;
  --color-danger: #b91c1c;

  /* Type */
  --font-sans: "Your Sans", system-ui, sans-serif;
  --font-mono: "Your Mono", ui-monospace, monospace;
  --text-xs: 0.75rem;   --text-sm: 0.875rem;  --text-base: 1rem;
  --text-lg: 1.125rem;  --text-xl: 1.25rem;   --text-2xl: 1.5rem;
  --text-3xl: 1.875rem; --text-4xl: 2.25rem;  --text-display: clamp(2.5rem, 5vw + 1rem, 4.5rem);
  --leading-tight: 1.2; --leading-body: 1.5;
  --tracking-display: -0.02em;

  /* Space: 4px base, with middle steps */
  --space-1: 0.25rem; --space-2: 0.5rem; --space-3: 0.75rem; --space-4: 1rem;
  --space-6: 1.5rem;  --space-8: 2rem;   --space-12: 3rem;   --space-16: 4rem;

  /* Radius: by level */
  --radius-control: 0.375rem;   /* inputs, buttons */
  --radius-container: 0.75rem;  /* cards, dialogs */
  --radius-pill: 999px;         /* badges only */

  /* Elevation */
  --shadow-raised: 0 1px 2px rgb(0 0 0 / 0.06);
  --shadow-overlay: 0 8px 24px -4px rgb(0 0 0 / 0.16), 0 2px 6px rgb(0 0 0 / 0.08);

  /* Motion */
  --ease-out: cubic-bezier(0.23, 1, 0.32, 1);
  --ease-in-out: cubic-bezier(0.77, 0, 0.175, 1);
  --duration-fast: 120ms; --duration-base: 200ms; --duration-slow: 360ms;

  /* Layout */
  --container-prose: 68ch;
  --container-page: 1200px;
}

@media (prefers-color-scheme: dark) {
  :root:not([data-theme="light"]) {
    --color-bg: #0c0a09;
    --color-surface: #1c1917;
    --color-surface-raised: #292524;
    --color-text: #fafaf9;
    --color-text-muted: #a8a29e;
    --color-border: #3f3a37;
    --color-accent: #7ea2ff;
    --color-accent-text: #0c0a09;
    --color-focus-ring: #7ea2ff;
  }
}
:root[data-theme="dark"] { /* same overrides as above, for explicit toggle */ }
```

Notes on that set:
- Dark theme is a separate designed palette: lighter accents, lower-intensity text, elevation via
  lighter surfaces. Check every pair in both themes.
- Redefine the dark values under both the media query (guarded by `:root:not([data-theme="light"])`)
  and `:root[data-theme="dark"]` so the manual toggle wins.
- Set `color-scheme` to match the active theme.
- The example values are placeholders. Replace them with the project's palette.

## Tailwind mapping

Map semantic tokens into Tailwind so classes read as roles, not values:

```css
/* Tailwind v4 */
@theme {
  --color-surface: var(--color-surface);
  --color-text: var(--color-text);
  --color-accent: var(--color-accent);
  --radius-control: var(--radius-control);
}
```

In Tailwind v3, extend `theme.colors` with `var(--…)` references. Keep components using
`bg-surface`, `text-muted`, `border-border`, not `bg-stone-100`.

## Variants

- Define variants as a fixed set (`primary`, `secondary`, `ghost`, `danger`) and sizes
  (`sm`, `md`, `lg`) with a single mapping function or a `class-variance-authority`-style helper
  that the project already uses.
- Every variant defines every state: default, hover, focus-visible, active, disabled, loading.
- Variants change appearance. If two variants differ in structure, make two components.
- Avoid boolean prop explosions (`isPrimary`, `isLarge`, `isOutline`); use a single `variant` and
  `size`.

## Component states checklist

For each interactive component, define and visually verify:

- default, hover (gated for touch), focus-visible, active/pressed
- disabled (native `disabled`, reduced emphasis, no pointer events)
- loading (same size, progress indicator, `aria-busy`)
- error and invalid (where applicable)
- selected / current / expanded where applicable
- dark theme for each of the above

## Primitives to build first

Button, link, input, textarea, select/combobox, checkbox, radio, switch, field (label + control +
help + error), dialog, popover/menu, tooltip, tabs, table, badge, toast, skeleton, empty state,
page header, section, container/grid. Pages are compositions of these; avoid one-off layouts.

## Documentation

- Document each token group with its intended use and a contrast note.
- Keep a single `DESIGN.md` (or the project's docs equivalent) only if the project will keep it
  current. Stale design docs are worse than none.

## Changing the system

- A new token requires a use case in at least one component today.
- Renaming tokens is a breaking change; deprecate with an alias first.
- Changing a semantic token's value updates every component; check both themes and all screens
  afterward (see `visual-qa.md`).

## Verify (design system)

- No raw hex values or `px` spacing in component files (search the diff).
- Every semantic token used resolves in light and dark.
- Every component's states exist and render.
- The same concept uses the same token everywhere (for example, all errors use `--color-danger`).
