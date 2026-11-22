# Component Ecosystem

Load when choosing or integrating a component library, icon set, animation library, or
inspiration source. Rule zero: **inspect the project's existing dependencies first, and do not
install a library just because it exists.**

## Procedure

1. Read `package.json` (and lockfile) for: UI libraries, icon sets, animation libraries, styling
   system, form and data libraries.
2. Check whether the need is already covered. If a Button, Dialog or Combobox exists in the project,
   use it or extend it.
3. If nothing covers it, pick the smallest option that fits the framework and version. Verify
   compatibility with the framework version (React 18 vs 19, Next.js App Router, Vue 3, Svelte 5,
   Tailwind v3 vs v4) through the library's current docs (context7 or the vendor docs).
4. Install only what is used. Avoid duplicate functionality (two date pickers, two toast systems,
   two icon sets).

## Options and when to use them

| Option | What it is | Use when | Watch out for |
|---|---|---|---|
| **Tailwind CSS** | Utility CSS | Project already uses it, or greenfield with a token system | Class soup; use semantic tokens and extract components, not repeated long strings |
| **shadcn/ui** | Copy-in components on Radix + Tailwind | React/Next projects that want owned, editable components | Copied code is yours to maintain; keep the variant system consistent |
| **Radix UI** (primitives) | Unstyled accessible primitives | Dialogs, menus, popovers, tabs, selects, tooltips where behavior must be right | Styling is your job; this is the behavior layer, not a look |
| **React Aria / Ariakit / Headless UI** | Accessible headless primitives | Alternative to Radix; Headless UI pairs with Tailwind/Vue | Pick one primitive family per project |
| **Motion** (motion.dev, formerly Framer Motion) | React/vanilla animation | Springs, gestures, layout and exit animations, interruptible motion | Layout animations run on the main thread; prefer CSS for busy-page transitions |
| **GSAP** | Timeline animation | Complex scroll-linked or timeline choreography | Licensing for some plugins; keep out of simple UI |
| **Lucide** | Icon set | Default icon library for consistent stroke icons | Import per icon, not the barrel |
| **Phosphor / Heroicons** | Icon sets | Alternatives to Lucide; match the project's existing set | Do not mix sets in one surface |
| **21st.dev** | Community component catalogue | Inspiration and copy-in starting points | Quality varies; verify accessibility, states, dark mode and tokens before adopting; adapt to the design system |
| **Magic UI** | Animated component collection (React + Tailwind) | Specific effects (marquee, shimmer buttons, animated beams) when the brief wants them | Effects are easy to overuse; apply the motion rules and reduced-motion path |
| **Aceternity UI** | Showcase-style animated components | Landing-page flourishes when the brief calls for spectacle | Heavy; check bundle cost and mobile performance; avoid as a default look |
| **TanStack Table / Virtual** | Headless table and virtualization | Complex data grids and long lists | Headless: you style it |
| **TanStack Query / SWR** | Server state | Client data caching and revalidation | Do not duplicate server state in global stores |
| **react-hook-form + zod** (or the project's equivalent) | Forms and validation | Forms with many fields and validation rules | Use the project's existing choice |
| **next/image, next/font** (Next.js) | Built-in image and font optimization | Next.js projects | Use them instead of hand-rolled equivalents |
| **Recharts / Chart.js / visx / ECharts** | Charts | Dashboard charts | Match the project's existing choice; keep chart colors semantic and labeled |

## Framework notes

- **React / Next.js:** App Router server components by default; client only at leaves. Verify
  each library is compatible with RSC if used in server components.
- **Vue / Nuxt:** prefer Nuxt UI or Reka UI (Radix-equivalent for Vue) when Radix is unavailable.
  Use `@vueuse/motion` or CSS for motion.
- **Svelte / SvelteKit:** use Svelte transitions and `animate:` directives; bits-ui for headless
  primitives.
- **Angular:** Angular CDK for behavior primitives; Angular animations (or CSS) for motion.
- **Native (SwiftUI, Jetpack Compose, Flutter, React Native):** use the platform's components and
  motion first; follow platform guidance (Human Interface Guidelines, Material 3); do not apply
  web-only tooling.

## Adoption checklist for any third-party component

- [ ] Compatible with the project's framework and version.
- [ ] Keyboard and screen-reader behavior verified (not assumed).
- [ ] All states implemented: focus, disabled, loading, error, empty.
- [ ] Respects `prefers-reduced-motion` if it animates.
- [ ] Uses or can be mapped to the project's design tokens (no foreign palette).
- [ ] Adds acceptable bundle weight; lazy-load if large.
- [ ] License compatible with the project.
- [ ] Does not duplicate something the project already has.

## Icons

- One library, one stroke width, one size scale (`icon-sm` 16, `icon-md` 20, `icon-lg` 24).
- Decorative icons: `aria-hidden="true"`. Meaningful icons: accessible name on the control.
- No emoji as UI icons. Brand logos: official assets with correct clear space and proportions.

## Inspiration (not copy sources)

Use these to study patterns, then rebuild in the project's own components and tokens:
21st.dev, Magic UI, Aceternity UI, component galleries of Linear, Vercel, Raycast, Stripe, and
Framer. Never copy another product's branding, proprietary copy, imagery, or distinctive layout
verbatim.
