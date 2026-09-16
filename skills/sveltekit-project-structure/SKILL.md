---
name: sveltekit-project-structure
description: Organize, review, or refactor SvelteKit and TypeScript projects by feature ownership, runtime boundary, dependency direction, and narrow public APIs. Use when deciding where routes, components, browser clients, shared contracts, server services, internal shared services, external-service adapters, database code, repositories, Web Workers, commands, tests, types, configuration, or assets belong; standardizing feature database access; proposing a project tree; tracing architectural dependencies; or correcting an existing structure.
---

# SvelteKit Project Structure

Make file placement and dependency decisions explicit. Favor incremental, behavior-preserving improvements over speculative directory trees or repository-wide moves.

## Workflow

1. Inspect local instructions, the existing tree, aliases, framework versions, package scripts, and relevant files. Treat repository rules as authoritative.
2. Read only the relevant sections of [references/style-guide.md](references/style-guide.md) using the routing table below. Read the complete guide only for a repository-wide architecture review.
3. When the task involves feature database access, repository design, direct schema or database-client imports, transaction ownership, or persistence boundaries in routes and services, read [references/feature-data-access.md](references/feature-data-access.md) completely.
4. Identify the narrowest true owner: product capability, internal shared service, external-service
   adapter, generic primitive, framework adapter, long-running executable, or one-shot command.
   A cohesive application capability with reusable contracts consumed by several features can be
   an internal shared service without owning SQL tables or a process. Reuse alone does not change
   the ownership of product-specific behavior.
5. Classify every affected module as browser-only, server-only, or deliberately cross-runtime.
6. Trace existing callers and dependency direction before moving or extracting code. Follow the repository's required impact-analysis workflow. Shared services never import product features; features
   supply named provider implementations through the composition root. External-service adapters
   own protocol, signing, and credentials but never business state.
7. Keep types, tests, helpers, fixtures, and assets with their narrowest owner. Promote them only
   when multiple owners genuinely share the same meaning.
   For nested feature families, distinguish parent-owned behavior from child public APIs; nesting
   does not grant unrestricted access to sibling internals.
8. Keep routes and executable entrypoints focused, but do not make them empty pass-through wrappers.
   Keep route-specific page composition, state, and markup in the route. Move reusable behavior
   behind owned feature, shared-service, or primitive APIs, and extract UI only when the extracted
   unit has a distinct responsibility or more than one real consumer.
9. Preserve behavior unless redesign is explicit, then run the repository's type checks and focused tests.
   When defining or changing architectural boundaries, verify the affected import/export rules
   with repository-native checks; report any enforcement gaps rather than claiming full coverage.
10. Report ownership decisions, runtime boundaries, dependency constraints, verification, and intentional deviations.

## Reference routing

Read the named sections in [references/style-guide.md](references/style-guide.md) through the next
same-level heading:

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
