---
name: portable-svelte-components
description: Implement reusable, data-driven Svelte 5 components from Figma or other visual designs by separating content, behavior, presentation, assets, and typed public contracts. Use when translating a landing section or composed design into Svelte, making a component reusable across pages or projects, extracting page-specific content into configuration, or extending a reusable component family. Do not use for unrelated everyday component edits.
---

# Portable Svelte Components

Build design-driven components whose consumers can supply typed content, assets, links, and intentional visual variants without copying markup.

## Workflow

1. Inspect local instructions, framework versions, the global stylesheet, design tokens, neighboring reusable components, public exports, consuming routes, and assets.
2. Read [references/portable-component-design.md](references/portable-component-design.md) completely.
3. When working from Figma, load the available Figma design-to-code instructions before requesting design context. Use the exact node and inspect both visual and structural context.
4. Define the narrowest reusable boundary and a small typed contract. Separate stable component defaults from page-specific content.
5. Keep behavior generic: accept data and callbacks instead of importing product DTOs, endpoints, or persistence logic.
6. Implement responsive, touch, keyboard, focus, screen-reader, and asset behavior as part of the component contract.
7. Follow required impact analysis, then run formatting, type checks, focused tests, and a visual review when possible.
8. Report contract choices, asset assumptions, responsive behavior, and verification.

## Boundaries

- Preserve repository-native Svelte syntax, styling, tokens, aliases, and public API conventions.
- Do not invent placeholder assets, replacement SVG paths, or short-lived production URLs.
- Do not make every visual value configurable; expose only meaningful content or design variants.
- Use `$svelte-component-development` for general component mechanics and `$sveltekit-project-structure` for broader ownership decisions when those skills are available.
