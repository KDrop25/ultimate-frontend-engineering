# React and Frontend Architecture

Load for component design, composition, state, data fetching, Server/Client split, Next.js
structure, hydration, bundle size or rendering performance. Works across React frameworks;
Next.js-specific notes are marked.

## Principles

- Follow the project's existing architecture. Introduce a pattern only for a concrete problem
  the project has today.
- Simplest structure that works. A component with three props does not need a compound API.
- Components do one thing. Separate data fetching (containers or hooks) from presentation.
- Colocate what changes together: a component, its hook, its types, its tests.
- Split a component when it has several reasons to change, or it grows past ~200 lines
  with multiple responsibilities. Length alone is not the reason.

## Composition over configuration

Avoid boolean prop sprawl. Each boolean doubles the possible states and hides the structure.

```tsx
// Avoid: configuration
<Card title="Tasks" showFooter headerVariant="large" footer={<Actions/>} />

// Prefer: composition
<Card>
  <Card.Header><Card.Title>Tasks</Card.Title></Card.Header>
  <Card.Body><TaskList tasks={tasks} /></Card.Body>
  <Card.Footer><Actions /></Card.Footer>
</Card>
```

Rules:
- **Avoid boolean props that customize behavior.** Replace `isCompact`, `showIcon`, `isEditable`
  with composed children or explicit variant components (`ButtonPrimary`, `ButtonGhost`) when the
  variants differ in structure, not just style. Style-only variants can stay as a `variant` prop.
- **Compound components** for complex widgets (tabs, menus, dialogs, composers): a provider holds
  state; subcomponents read it through context; consumers compose the parts they need.
- **Children over render props.** Use `children` for composition. Use `renderX` props only when
  the parent must pass data back into the render slot.
- **Explicit variants** over mode booleans: `<Composer.Input />` and `<Composer.Textarea />`
  instead of `<Composer multiline />`.
- **React 19:** `use()` replaces `useContext()`; `forwardRef` is no longer required because `ref`
  is a prop. Skip these notes on React 18 or earlier.

## State management

Choose the simplest option that fits:

| Need | Tool |
|---|---|
| Local UI state | `useState` |
| Shared by 2–3 siblings | Lift state to the parent |
| Theme, locale, auth (read-heavy) | Context |
| Filters, pagination, selected tab, shareable state | URL search params |
| Server data with caching and revalidation | TanStack Query, SWR, or framework data APIs |
| Complex client-only state shared app-wide | Zustand or similar small store |

- Decouple UI from state implementation: the provider is the only place that knows where state
  lives; components consume a stable interface (`state`, `actions`, `meta`). This lets the same
  UI run on local state or synced global state.
- Avoid prop drilling deeper than ~3 levels: use context or restructure.
- Derive values during render instead of syncing them in `useEffect`.
- Use functional `setState` updates when the next value depends on the previous one, so
  callbacks stay stable.
- Keep transient, high-frequency values (pointer position, scroll offsets) out of state; use
  refs or motion values.

## Data fetching

- Fetch on the server where the framework allows (RSC, route loaders). Pass only the fields the
  client needs.
- Start independent requests together: `Promise.all([...])`, not sequential `await`s. Move
  `await` into the branch where the value is used.
- Avoid waterfalls: fetch the parent's data and the child's data in parallel, or preload.
- Client fetching: use a caching library for deduplication and revalidation. Do not write ad hoc
  `useEffect` fetch loops.
- Handle the three states for every query: loading (skeleton), error (retry), empty (empty state).
- Optimistic updates for low-risk writes, with rollback on error.

## Server and Client components (Next.js App Router and similar)

- Default to server components. Add `"use client"` only where you need state, effects, browser
  APIs or event handlers, and push the client boundary down to the smallest leaf.
- Do not pass large serialized objects to client components. Pass ids or the trimmed fields the
  client uses.
- Authenticate server actions like API routes; they are public endpoints.
- Use `React.cache()` for per-request deduplication of server data; use an LRU or framework cache
  for cross-request data.
- Do not keep mutable module-level state for per-request data in server code.

## Suspense and streaming

- Wrap slow, independent regions in `Suspense` with a skeleton that matches their final shape.
  Stream the shell first.
- Place boundaries around independent data, not around every component.
- Error boundaries around regions that can fail independently, so one failing widget does not
  blank the page.

## Code splitting and lazy loading

- Load heavy, below-the-fold or rarely used components with `next/dynamic` or `React.lazy`:
  charts, editors, maps, PDF viewers, modals with large content.
- Load code on intent: preload on hover or focus of the trigger.
- Load third-party scripts (analytics, chat, A/B) after hydration or on idle.
- Load features only when activated: `if (flagEnabled) import(...)`.

## Bundle hygiene

- Import directly from the module, not from barrel files of large libraries
  (`import { Check } from "lucide-react/dist/esm/icons/check"` or the package's documented
  per-icon import; check the library's current guidance via docs).
- Keep paths statically analyzable (no template-built import paths) so bundlers can split them.
- Prefer lightweight equivalents when they cover the need. Measure before and after.

## Re-render discipline

Measure first with React DevTools Profiler. Then:

- Move expensive subtrees into memoized components so early returns skip the work.
- Use `memo` on components with stable props that re-render often under a changing parent.
- Use `useMemo`/`useCallback` only for measured expensive computations or referential stability
  that a memoized child depends on. Do not wrap everything.
- Subscribe to derived booleans, not raw values (`isOverflowing`, not `scrollTop`).
- Use primitive dependencies in effects. Put interaction logic in event handlers, not effects.
- Keep `useEffectEvent` (or the stable-ref pattern) results out of effect dependency arrays.
- Defer non-urgent updates with `startTransition` or `useDeferredValue` (search results, large
  filters), keeping input responsive.
- Never define components inside components: each render creates a new type and remounts it.
- Hoist static JSX and constant objects outside components.
- Use `Activity` (or `display`) to preserve state when hiding, instead of unmounting.
- Use a ternary instead of `&&` when the left side can be `0` or `""`, which renders as text.
- React Compiler: if the project enables it, skip manual `memo`/`useMemo`/`useCallback`.

## Hydration

- Render the same markup on server and client. Guard browser-only values (`window`, time zones,
  `Date.now()`, locale-formatted values) with an effect or a client-only component.
- Use `suppressHydrationWarning` only on the exact element where a mismatch is expected
  (timestamps), never on containers.
- For client-only data needed before paint (theme), use a small inline script in `<head>` to set
  the attribute before hydration to avoid flicker.
- Controlled inputs need `onChange`; use `defaultValue` for uncontrolled fields.

## Component API design

- Props named by meaning (`onSelect`, `isDisabled`), not implementation.
- Accept `className` and spread remaining native props on primitives so consumers can extend.
- Forward `ref` on primitives that wrap DOM elements.
- Keep controlled and uncontrolled modes consistent (`value`/`defaultValue`).
- Stable keys: use ids from data, never array indices for reorderable lists.

## Testing

- Test behavior through the user's interaction (role, label, text), not implementation details.
- Visual checks are separate (see `visual-qa.md`); unit tests do not prove layout.
- Keep the test that fails when the logic breaks; skip tests that only restate the markup.

## Verify (architecture)

- Each component has one clear purpose and a typed, minimal prop surface.
- No boolean props that change structure.
- Client boundary is as low in the tree as it can be.
- No waterfalls on the critical path.
- No unbounded lists rendered without pagination or virtualization.
