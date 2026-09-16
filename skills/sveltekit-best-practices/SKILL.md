---
name: sveltekit-best-practices
description: Implement, review, organize, or refactor Svelte 5 and SvelteKit applications using clear state ownership, explicit runtime boundaries, accessible component patterns, focused routes, feature-oriented structure, narrow public APIs, and repository-native conventions. Use for component and route implementation, runes and effects, SSR and hydration, forms and interactions, project structure, feature ownership, services, repositories, database access, dependency direction, or Svelte-focused code review. Do not use solely for design-to-code portability or non-Svelte TypeScript work.
---

# SvelteKit Best Practices

Build and organize SvelteKit code with explicit ownership of behavior, state, runtime boundaries, and dependencies. Favor focused, behavior-preserving improvements over mechanical abstractions or speculative restructuring.

## Workflow

1. Inspect local instructions, framework versions, the existing tree, aliases, styles, package scripts, tests, and relevant files. Treat repository rules as authoritative.
2. Read only the references required for the task using the routing table below. Read a selected reference completely; read the full structure guide only for a repository-wide review.
3. Define each affected component or module's responsibility, state ownership, public contract, and runtime boundary before changing it.
4. Keep deterministic transformations pure and make data-retrieval helpers return typed results. Event handlers and route or component orchestrators may deliberately update state owned by that component when coordinating loading, errors, cancellation, progressive results, focus, or other UI lifecycle concerns. Do not hide mutation of caller-owned or module-global state behind helper functions.
5. For structural decisions, identify the narrowest true owner: product capability, internal shared service, external-service
   adapter, generic primitive, framework adapter, long-running executable, or one-shot command.
   A cohesive application capability with reusable contracts consumed by several features can be
   an internal shared service without owning SQL tables or a process. Reuse alone does not change
   the ownership of product-specific behavior.
6. Classify every affected module as browser-only, server-only, or deliberately cross-runtime.
7. Trace existing callers and dependency direction before editing, moving, or extracting code. Follow the repository's required impact-analysis workflow. Shared services never import product features; features
   supply named provider implementations through the composition root. External-service adapters
   own protocol, signing, and credentials but never business state.
8. Keep types, tests, helpers, fixtures, and assets with their narrowest owner. Promote them only
   when multiple owners genuinely share the same meaning.
   For nested feature families, distinguish parent-owned behavior from child public APIs; nesting
   does not grant unrestricted access to sibling internals.
9. Keep routes and executable entrypoints focused, but do not make them empty pass-through wrappers.
   Keep route-specific page composition, state, and markup in the route. Move reusable behavior
   behind owned feature, shared-service, or primitive APIs, and extract UI only when the extracted
   unit has a distinct responsibility or more than one real consumer.
10. Preserve behavior unless redesign is explicit, then run the repository's formatter, type checks, focused tests, and visual or accessibility checks proportional to the change.
   When defining or changing architectural boundaries, verify the affected import/export rules
   with repository-native checks; report any enforcement gaps rather than claiming full coverage.
11. Report state and ownership decisions, runtime boundaries, interaction or accessibility considerations, verification, and intentional deviations.

## Reference routing

Select the relevant references below. When an entry names sections in
[references/style-guide.md](references/style-guide.md), read each named section through the next
same-level heading:

- Everyday components, runes, effects, SSR, hydration, markup, accessibility, forms, and interaction behavior: read [references/svelte-best-practices.md](references/svelte-best-practices.md) completely.
- Shared services, provider contracts, external-service adapters, and the composition root: `## Internal shared services`, `## External-service adapters`, and `## The three meanings of "service"`.
- Services and CLI commands: `## Standalone services and commands`.
- Overall architecture or target trees: `## Purpose`, `## Architectural vocabulary`, `## Dependency direction`, and `## Feature boundaries`.
- Feature families, nested capabilities, parent APIs, and sibling dependencies: read [references/feature-families.md](references/feature-families.md).
- Automated architectural checks, public-surface rules, import restrictions, and cycles: read [references/boundary-enforcement.md](references/boundary-enforcement.md).
- Components, types, helpers, and public exports: `## Generic UI ownership` through `## Public APIs and index.ts`.
- Reactive state or browser access: `## Reactive state and functions`, `## Browser data access`, and `## Browser workers`.
- Routes and APIs: `## Route responsibilities` and `## Validation and network boundaries`.
- Auth, database infrastructure, storage, and migrations: `## Authentication, database, and storage primitives` and `## Database schemas and migrations`.
- Feature repositories and persistence access: read [references/feature-data-access.md](references/feature-data-access.md) completely.
- Naming, tests, framework roots, artifacts, or environment: read the matching heading near the end of the guide.
- Final architecture review: `## Review checklist` and `## Adoption`.

## Core constraints

- Use `onMount` for setup that occurs once after mounting and return cleanup for persistent resources. Use `$effect` only for synchronization that must rerun with reactive dependencies.
- Treat ordinary component `<script>` state as instance-owned, not global. Avoid mutable module-level state unless it is an intentional shared store with a documented lifecycle.
- Prefer returned values for pure helpers and data retrieval. Allow deliberate local state mutation in clearly named event handlers and UI lifecycle orchestrators; separate fetching or transformation when that makes either part independently testable.
- Use `$state.snapshot(...)` before cloning, serializing, or passing rune state across worker, network, or persistence boundaries.
- Keep product API calls and persistence out of presentational components. Keep loading and error state close to the request owner.
- Preserve narrow ownership, explicit runtime boundaries, inward dependencies, thin adapters, typed contracts, and colocated tests.
- Internal shared services expose typed public contracts and never import product features; feature-specific behavior crosses the boundary as named provider implementations registered by the composition root.
- External-service adapters own protocol, signing, verification, and credential injection; business state such as orders, retries, and durable message intent stays in the consuming feature.
- Adapt examples and aliases to the target repository. Do not introduce a tool or directory solely because it appears in the guide.
- Do not create empty directories, barrels, type files, or helpers merely to match an example tree.
- Do not move an entire route into a single-use `*Page.svelte` component merely to make the route
  file shorter. File length alone does not justify a component boundary.
- Do not hide sideways feature dependencies behind generic `shared`, `common`, or `utils` modules.
- Keep product queries in feature-owned repositories. Keep connections, transactions, schema definitions, and cross-feature mechanics in the database primitive.
- Restrict direct ORM schema imports to repository modules and application database-singleton imports to runtime composition roots, except for documented infrastructure integrations.
- Derive target structures from actual capabilities and runtimes rather than copying example names.
