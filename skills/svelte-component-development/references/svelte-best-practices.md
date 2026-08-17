# General Svelte Best Practices

This guide covers day-to-day Svelte and SvelteKit implementation practices. Adapt the examples to
the repository's installed Svelte version, compiler settings, styling system, and local conventions.
Use the repository's architecture guidance, or `$sveltekit-project-structure` when available, for
file ownership, dependency direction, and project structure.

## Contents

- [Component design](#component-design)
- [Runes and reactivity](#runes-and-reactivity)
- [SSR, hydration, and browser boundaries](#ssr-hydration-and-browser-boundaries)
- [Markup, accessibility, and styling](#markup-accessibility-and-styling)
- [Forms, data, and errors](#forms-data-and-errors)
- [Performance and verification](#performance-and-verification)

Use the repository's installed Svelte and SvelteKit versions, compiler settings, and established
syntax conventions. Do not introduce a new Svelte pattern merely because it is newer than the
project's current version.

## Component design

- Give each component one clear responsibility and keep route components focused on composition.
- Define an explicit typed prop contract with `$props()` in Svelte 5. Keep component-private types
  in the component; move externally constructed contracts to the nearest component package.
- Prefer callbacks or explicit component events over hidden mutation of parent state. Keep data
  transformations and state transitions in pure TypeScript functions when they can be tested without
  rendering.
- Use snippets and component composition for reusable markup. Do not create a wrapper component
  solely to shorten a template unless it owns behavior, styling, or a meaningful public contract.
- Keep DOM and browser API work explicit. Use `onMount` instead of `$effect` for setup that runs once
  after mounting and return cleanup for listeners, observers, timers, subscriptions, and other
  persistent resources.
- Use `$effect` sparingly. Prefer updating state from events to reduce implicit state updates.

## Runes and reactivity

- Use `$state` for local mutable state, `$derived` for values computed from state, and `$effect`
  only to synchronize with an external system such as the DOM, a browser API, or an imperative
  library.
- Prefer derived values and event handlers over effects that copy state, trigger business logic, or
  mirror one variable into another. Effects should be small and have an obvious cleanup boundary.
- Pass reactive inputs explicitly to helpers. Avoid module-level mutable state unless it is an
  intentional shared store with documented ownership and lifecycle.
- Snapshot rune proxies before cloning or serializing them. Do not pass live reactive proxies across
  network, worker, or persistence boundaries.
- Treat derived values as computed state rather than independent mutable state. If a project intentionally
  overrides a derived binding, follow its established convention and make that behavior explicit.

## SSR, hydration, and browser boundaries

- Assume component code can run during SSR. Guard `window`, `document`, `localStorage`, media
  queries, layout measurement, and other browser-only APIs behind a browser boundary or `onMount`.
- Keep initial server-rendered markup deterministic so hydration sees the same structure and content.
  Defer client-only values such as viewport dimensions and random IDs until the client owns them.
- Put secrets and private environment imports in server modules only. Public environment values are
  not a substitute for authorization or secret storage.
- Clean up every resource created by a component, including event listeners, observers, workers,
  timers, and subscriptions.

## Markup, accessibility, and styling

- Prefer semantic HTML and native controls before adding custom interaction. Every form control needs
  an associated label; interactive elements need an accessible name, keyboard behavior, and visible
  focus treatment.
- Preserve heading order, meaningful landmarks, sufficient color contrast, and useful loading,
  empty, error, and disabled states. Do not use ARIA to repair an element that should be native.
- Keep feature styles with feature components and generic tokens/resets in the configured global
  stylesheet. Follow the project's existing CSS or utility-class system instead of mixing systems.
- Avoid coupling presentational components to API calls, database models, or product-specific
  mutations. Adapt feature data at the feature boundary and pass a generic UI contract inward.

## Forms, data, and errors

- Treat form fields, URL parameters, action inputs, and API responses as untrusted data. Validate at
  the server boundary and provide typed, field-specific feedback in the UI.
- Use SvelteKit form actions and progressive enhancement when they fit the feature. Keep pending,
  success, validation-error, and unexpected-error states visible and distinct.
- Keep loading and error state close to the request owner. Do not scatter fetch calls or response
  parsing through presentational components.

## Performance and verification

- Measure before optimizing. Prefer smaller component boundaries, stable keyed lists, and avoiding
  unnecessary effects before introducing memoization or custom caching.
- Use `{#each items as item (item.id)}` for collections whose identity matters. Do not use array
  indexes as keys when items can be inserted, removed, or reordered.
- Test pure state transitions as functions, component tests through observable behavior, and route
  or server behavior at its transport boundary. Run the repository's formatter, type checker, and
  focused tests after implementation.
