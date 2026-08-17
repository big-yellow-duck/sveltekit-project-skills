# Portable Component Design

Build reusable, data-driven Svelte 5 components that separate content, behavior, presentation, and
assets. A consuming page should be able to supply typed content, links, assets, and optional visual
overrides without copying the component markup. Follow the target repository's ownership and
general Svelte implementation guidance when it is stricter or more specific.

## Workflow

1. Inspect the target repository before editing. Read the configured global stylesheet for tokens,
   fonts, breakpoints, and base rules. Inspect nearby reusable components, their public types and
   barrel exports, and the consuming route.
2. For Figma implementation, load and follow the repository's Figma design-to-code guidance before
   requesting design context. Use the exact node-specific URL, study both the screenshot and design
   context, and treat generated React/Tailwind as visual reference only. Convert the result to Svelte
   5 and the repository's conventions.
3. Run the repository's required impact analysis before modifying an existing function, class, or
   method. Prefer additive component work when the call graph is unavailable or stale, and report an
   unknown blast radius rather than inventing one.
4. Define a small `types.ts` contract. Pass repeated content as arrays of groups, items, or cards.
   Keep stable design defaults near the component; move page-specific copy and links to the consuming
   route or a content module.
5. Export the component and its public types from `index.ts` when it has a meaningful public surface.
   Use `$props`, `$state`, `$derived`, and ordinary typed functions in the project's Svelte 5 style.
6. Keep assets under the narrowest owner. For Figma assets, use exported project assets rather than
   short-lived Figma URLs. Never hand-author replacement SVG paths or leave visual placeholders in a
   finished implementation.

## Styling and responsive behavior

- Use the project's existing utility classes and design tokens first. Do not install another styling
  system or introduce duplicate tokens.
- Keep responsive behavior in the component using the project's breakpoints. Horizontal designs
  should support touch scrolling, keyboard focus, accessible labels, and controls that actually move
  the scroll container.
- If existing utilities and tokens are insufficient, ask before adding extra styles. After approval,
  keep new CSS scoped to the component and explain why utilities could not express the design.
- Give images explicit sizing behavior, preserve aspect ratio, and use meaningful alt text for
  content images. Mark decorative assets as hidden from assistive technology.
- Follow the repository's Svelte ordering convention: imports, props/state, derived values/functions,
  then effects/lifecycle. Snapshot rune state before cloning; do not use `structuredClone` on live
  rune proxies.

## Validation and handoff

Run the repository's type check, formatter/linter, and focused component or landing tests before
handoff. Inspect the diff for unrelated changes, confirm new assets are present, and mention any
design-asset, responsive-behavior, or visual-review assumptions.
