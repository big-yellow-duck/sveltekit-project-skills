---
name: sveltekit-project-structure
description: Organize, review, or refactor SvelteKit and TypeScript projects by feature ownership, runtime boundary, dependency direction, and narrow public APIs. Use when deciding where routes, components, browser clients, shared contracts, server services, database code, Web Workers, commands, tests, types, configuration, or assets belong; proposing a project tree; tracing architectural dependencies; or correcting an existing structure.
---

# SvelteKit Project Structure

Make file placement and dependency decisions explicit. Favor incremental, behavior-preserving improvements over speculative directory trees or repository-wide moves.

## Workflow

1. Inspect local instructions, the existing tree, aliases, framework versions, package scripts, and relevant files. Treat repository rules as authoritative.
2. Read only the relevant sections of [references/style-guide.md](references/style-guide.md) using the routing table below. Read the complete guide only for a repository-wide architecture review.
3. Identify the narrowest true owner: product capability, generic primitive, framework adapter, long-running service, or one-shot command.
4. Classify every affected module as browser-only, server-only, or deliberately cross-runtime.
5. Trace existing callers and dependency direction before moving or extracting code. Follow the repository's required impact-analysis workflow.
6. Keep types, tests, helpers, fixtures, and assets with their narrowest owner. Promote them only when multiple owners genuinely share the same meaning.
7. Keep routes and executable entrypoints thin. Move reusable behavior behind owned feature or primitive APIs.
8. Preserve behavior unless redesign is explicit, then run the repository's type checks and focused tests.
9. Report ownership decisions, runtime boundaries, dependency constraints, verification, and intentional deviations.

## Reference routing

Search `references/style-guide.md` for these headings and read the applicable section through the next same-level heading:

- Overall architecture or target trees: `## Purpose`, `## Architectural vocabulary`, `## Dependency direction`, and `## Feature boundaries`.
- Components, types, helpers, and public exports: `## Generic UI ownership` through `## Public APIs and index.ts`.
- Reactive state or browser access: `## Reactive state and functions`, `## Browser data access`, and `## Browser workers`.
- Routes and APIs: `## Route responsibilities` and `## Validation and network boundaries`.
- Services and CLI commands: `## Standalone services and commands`.
- Auth, databases, storage, and migrations: `## Authentication, database, and storage primitives` and `## Database schemas and migrations`.
- Naming, tests, framework roots, artifacts, or environment: read the matching heading near the end of the guide.
- Final architecture review: `## Review checklist` and `## Adoption`.

## Core constraints

- Preserve narrow ownership, explicit runtime boundaries, inward dependencies, thin adapters, typed contracts, and colocated tests.
- Adapt examples and aliases to the target repository. Do not introduce a tool or directory solely because it appears in the guide.
- Do not create empty directories, barrels, type files, or helpers merely to match an example tree.
- Do not hide sideways feature dependencies behind generic `shared`, `common`, or `utils` modules.
- Derive target structures from actual capabilities and runtimes rather than copying example names.
