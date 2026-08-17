---
name: svelte-component-development
description: Implement or review Svelte 5 and SvelteKit components using explicit typed props, runes, SSR-safe browser boundaries, accessible markup, deliberate effects, colocated state logic, and repository-native styling. Use for ordinary Svelte component work, route UI implementation, forms, interaction behavior, accessibility fixes, hydration issues, or component-focused code review. Do not use solely for project-tree or server architecture decisions.
---

# Svelte Component Development

Build components that follow the target repository's installed Svelte version and local conventions.

## Workflow

1. Inspect local instructions, framework versions, neighboring components, global styles, aliases, scripts, and tests.
2. Read [references/svelte-best-practices.md](references/svelte-best-practices.md) completely before implementing or reviewing component code.
3. Define the component's responsibility, typed public contract, state ownership, and SSR/browser boundary.
4. Keep deterministic transformations and state transitions in typed functions when they can be tested without rendering.
5. Prefer semantic HTML, native controls, visible focus, keyboard access, and explicit loading, empty, error, and disabled states.
6. Use repository-native tokens and styling. Do not introduce another styling system or duplicate global tokens.
7. Follow required impact analysis before editing existing symbols, then run the formatter, type checker, and focused tests.
8. Report interaction, accessibility, hydration, and verification decisions.

## Boundaries

- Use `onMount` for setup that occurs once after mounting and return cleanup for persistent resources.
- Use `$effect` only for synchronization that must rerun as reactive dependencies change.
- Keep route components focused on composition and route-lifetime state.
- Keep product API calls and persistence out of presentational components.
- Use `$state.snapshot(...)` before cloning or serializing rune state.
- Defer project-wide placement and dependency decisions to `$sveltekit-project-structure` when both skills are available.
