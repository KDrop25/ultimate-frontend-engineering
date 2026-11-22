# Visual Design

Load when choosing colors, type, layout, depth, radii, or theme. Every rule has a reason; if the
reason does not apply to the brief, drop the rule.

## Design plan (write this before code)

- **Subject and audience:** what the product is, who uses it, in what setting (desk, commute,
  dim room, bright office). Light vs dark follows the use scene, not the category.
- **Palette:** 4–6 named hex values: background, surface, text, accent, plus semantic
  success/warning/danger. Give each a role.
- **Type:** one or two families with clearly different jobs; the role scale; the body measure.
- **Layout concept:** one sentence plus a rough ASCII sketch. Decide alignment (left is the
  default for reading content; center only for short, isolated moments).
- **Principles:** three short rules that would make this design recognizably itself.
- **Commit point:** name the single memorable element.

## Hierarchy

- Squint test: blur the screen. You should still see the primary element, the secondary
  element, and the groups, in that order.
- Size, weight, space and color each carry hierarchy. Use at least two of them per level.
  Do not rely on size alone.
- One primary action per view. Secondary actions are visually quieter (ghost or outline).
- Headings sit closer to the content they introduce than to whatever comes before them.

## Typography

- Pick the typeface from the product's subject and audience; color comes after and supports it.
- Body 16px minimum on web; prose measure 45–75ch (long-form reading sits at the 55–75 end).
  Serif body needs slightly more line height than sans.
- Line height: about 1.5 for body, tighter (1.0–1.2) for large display, and wider for light
  text on dark surfaces. Tune to the face, not a universal ratio.
- Display tracking: tighten toward -0.02 to -0.04em at large sizes; never looser than the
  base for display. Body tracking stays 0 unless the face needs help.
- `text-wrap: balance` on headings and short centered blocks; `text-wrap: pretty` on body.
- Use `font-variant-numeric: tabular-nums` for numeric columns and comparisons.
- Load only the weights used. Use `font-display: swap` and a metric-compatible fallback to
  avoid layout shift.
- Avoid: a single accented word in a headline (italic or colored), all-caps labels by default,
  monospace as a "technical" costume (use it for code and data only), system display fonts
  (Impact, Arial Black) as the voice of a designed page.

## Color

- Neutral base with one accent used for the primary action and key focus. A second accent
  only with a clear semantic job.
- Semantic tokens: `text-primary`, `text-muted`, `surface`, `surface-raised`, `border`,
  `accent`, `accent-foreground`, `danger`, `success`, `focus-ring`.
- Contrast: body text ≥ 4.5:1; large text and meaningful non-text UI ≥ 3:1. Tint secondary
  text from the surface hue rather than using neutral gray on colored backgrounds.
- Dark mode is designed, not inverted: reduce pure-white text intensity, lower saturation of
  accents, use elevation via lighter surfaces rather than shadows.
- Color is never the only signal for state. Pair with text or an icon.
- Avoid: near-black with a tint standing in for black, warm-cream-plus-terracotta as a default
  palette (a common AI-generated look), a single acid-green accent on black as a
  default.

## Spacing and layout

- A 4px base with an 8px rhythm for component and section spacing. A 4-step middle
  (4, 8, 12, 16) is usually needed; an 8-only scale is too coarse for dense UI.
- Define named section tiers (for example 16 / 24 / 32 / 48 / 64 / 96) and reuse them.
- Grid for page structure; flex for component internals. Prefer `gap` for sibling rhythm.
- Content width: prose ≤ 65–75ch; app content full width with sensible max widths
  (1200–1440px) for marketing.
- Left edges align across sections. Nothing is centered by accident.
- Watch selector specificity when stacking section padding and margins; conflicting
  `.section` and `.cta` rules are a common cause of wrong spacing.

## Radius, border and depth

- Radius tokens by level: `sm` for inputs and chips, `md` for controls, `lg` for cards and
  dialogs. Nested radii: inner radius = outer radius minus padding.
- One consistent border treatment per surface. Full borders or tinted surfaces; avoid a bar
  down one side of a rounded box.
- Shadows: offset and blur, low opacity. A zero-offset colored glow is decoration. Elevation
  tokens (`shadow-sm`, `shadow-md`) reserved for floating layers (menus, dialogs).
- Glass and blur only when they perform a specific job (separating a floating layer over
  content). Not as default decoration.

## Visual structure must encode information

- Cards are for grouping discrete items the user compares or acts on. Not as the default page
  structure. Never nest cards.
- Dividers, numbered steps, badges and eyebrow labels must correspond to real structure.
  Numbers only for real sequences.
- Icons are drawn from one library with one stroke weight and size scale. Decorative icons
  beside text are hidden from assistive tech.
- Imagery should carry real content. Use decorative shapes or abstract art only when they
  have a clear role; do not use them to fill space where content is missing.

## Styles as options (use only when the brief is open)

When direction is open, pick a language that fits the subject and commit to it. Common options
and their fit:

| Language | Fits | Watch out for |
|---|---|---|
| Swiss / Editorial | content, publishing, portfolios | sparse content looks unfinished |
| Modern corporate | B2B, enterprise, trust | becoming generic blue cards |
| Dark luxe | premium consumer, creative tools | low contrast on secondary text |
| Bento grid | feature overviews | uniform boxes that signal no priority |
| Brutalist / neo-brutalist | playful, opinionated brands | hard shadows as costume |
| Bauhaus / Memphis | art, education, bold brands | legibility under heavy decoration |
| Organic / Wabi-sabi | wellness, slow-living, food | softness that reads as unfinished |

Use a language's principles (grid, texture, color logic), not its clichés. Before committing,
name what would make the result a competitor's site if the logo were swapped; push the signature
element harder.

## Verify (visual)

- Hierarchy passes the squint test at each viewport.
- Spacing values come from tokens (search the diff for stray `px` values).
- Contrast passes at 4.5:1 for body text in both themes (check computed values).
- The memorable element is present and the rest of the page is quiet.
