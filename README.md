# Ultimate Frontend Engineering

A Claude Code skill for frontend work: websites, landing pages, SaaS apps, admin dashboards,
design systems, animation, React and Next.js architecture, performance, accessibility, and
visual QA. It activates on frontend tasks and loads reference files only when they are needed.

## What it covers

| Area | Reference |
|---|---|
| Visual design: hierarchy, type, color, spacing, radius, depth, themes | `references/visual-design.md` |
| UX: navigation, forms, tables, search, dashboards, states, copy | `references/ux-patterns.md` |
| Motion: when to animate, easing, springs, gestures, reduced motion | `references/motion.md` |
| React and Next.js: composition, state, data, Server/Client, bundles, re-renders | `references/react-architecture.md` |
| Performance: Core Web Vitals, images, fonts, rendering | `references/performance.md` |
| Accessibility: semantics, keyboard, ARIA, contrast, focus, zoom | `references/accessibility.md` |
| Design systems: tokens, theming, variants, states | `references/design-system.md` |
| Component ecosystem: Tailwind, shadcn/ui, Radix, Motion, icons, and more | `references/component-ecosystem.md` |
| Visual QA: browser loop, viewports, critique scores, release gates | `references/visual-qa.md` |
| Worked examples: SaaS dashboard, landing page, UI refinement | `references/examples.md` |

## Layout

```
ultimate-frontend-engineering/
├── SKILL.md          # entry point: non-negotiables, workflow, quick rules, reference index
├── README.md         # this file
├── LICENSE
└── references/       # loaded on demand, one topic each
    ├── visual-design.md
    ├── ux-patterns.md
    ├── motion.md
    ├── react-architecture.md
    ├── performance.md
    ├── accessibility.md
    ├── design-system.md
    ├── component-ecosystem.md
    ├── visual-qa.md
    └── examples.md
```

`SKILL.md` is the only file loaded automatically. It tells the agent which reference to read for
each task, so a small styling fix loads two or three files, and a full dashboard build can load more.

## Installation

Clone the repository into your Claude Code skills folder:

```bash
# macOS / Linux
git clone https://github.com/KDrop25/ultimate-frontend-engineering.git ~/.claude/skills/ultimate-frontend-engineering
```

```powershell
# Windows (PowerShell)
git clone https://github.com/KDrop25/ultimate-frontend-engineering.git "$HOME\.claude\skills\ultimate-frontend-engineering"
```

Claude Code discovers the skill from the `name` and `description` in `SKILL.md`. No configuration
is needed. To update later, run `git pull` inside that folder.

To confirm it loads, start a session and ask for a UI task, or invoke it by name:
`ultimate-frontend-engineering`.

## Usage

Describe the frontend task in plain language. Examples that should trigger it:

- "Build an admin dashboard for our billing team."
- "Make this landing page feel less generic."
- "Review my settings page for accessibility and performance."
- "Add motion to the onboarding flow, respecting reduced motion."
- "Create a design system with light and dark themes for this Next.js app."
- "Why does this list re-render on every keystroke?"

The workflow runs in order: discover the project, set design direction, plan architecture,
implement, verify visually, then polish. Visual verification uses a browser tool such as Playwright
or Claude in Chrome when one is available. Without one, the report says that visual checks were
not run.

## Design principles

1. Follow the project's existing conventions and the user's brief before any default.
2. Every decision has a reason. Remove anything that cannot justify itself.
3. Compiling is not done. Look at the rendered UI at phone, tablet, and desktop widths.
4. Accessibility and reduced motion are minimum requirements.
5. No speculative abstraction.

## Limitations

- Visual QA needs a browser tool. Without one, visual checks are not performed and the report says so.
- Performance numbers need measurement on target devices. The skill states when numbers were not measured.
- Guidance is opinionated about craft and accessibility. A brief that conflicts with those minimums is
  raised with the user rather than silently overridden.
- Coverage is strongest for web frontends. Native mobile guidance is limited to pointers in the
  component ecosystem reference.

## Contributing

Issues and pull requests are welcome. Keep `SKILL.md` short and move detail into `references/`.
Each reference file should stay focused on one topic and give actionable rules with a way to verify them.

## License

MIT. See [LICENSE](LICENSE).

All guidance in this repository is written for this project and does not reproduce text from
third-party publications. Product names (such as Linear or Stripe) appear only as design
benchmarks, and library names appear only where the guidance refers to them. Standard values
such as easing curves, WCAG thresholds, and Core Web Vitals targets are published facts and are
used as such. Verify any value against current documentation before relying on it in production.
