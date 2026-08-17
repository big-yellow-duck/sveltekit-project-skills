---
name: sveltekit-project-structure
description: Develop, organize, review, or refactor SvelteKit 5 and TypeScript projects by feature ownership, runtime boundary, dependency direction, accessible Svelte component practices, and narrow public APIs. Use when implementing general Svelte features, deciding where Svelte components, routes, browser clients, shared contracts, server services, database code, Web Workers, commands, tests, types, configuration, or assets belong; when proposing a project tree; or when correcting an existing SvelteKit structure without mixing browser, server, common, UI, and product-feature responsibilities.
---

# SvelteKit Project Structure

Apply the bundled tool-agnostic SvelteKit architecture guide to make file placement and dependency decisions explicit. Favor incremental, behavior-preserving improvements over speculative directory trees or repository-wide moves.

## Workflow

1. Read [references/style-guide.md](references/style-guide.md) completely for structure guidance and [references/svelte-best-practices.md](references/svelte-best-practices.md) when implementing or reviewing Svelte code.
2. Inspect the project's local instructions, existing tree, import aliases, SvelteKit version, package scripts, and relevant files. Treat repository-specific instructions as authoritative when they conflict with the reference.
3. For general Svelte implementation, preserve the component's SSR/browser boundary, use the project's Svelte version and event conventions, keep state transitions explicit, and verify keyboard and screen-reader behavior for interactive UI.
4. Identify the narrowest true owner of each behavior before choosing a directory:
   - product capability: browser `feature/<name>`, server `server/feature/<name>`, or cross-runtime `common/feature/<name>`
   - generic capability: UI, auth, database, storage, or another precisely named primitive
   - framework adapter: route, hook, page/layout load, or API handler
   - executable adapter: long-running `services/<name>` or one-shot `commands/<name>`
5. Classify the runtime boundary: browser-only, server-only, or deliberately browser/server-safe. Keep common code serializable, deterministic, and safe for browser bundles.
6. Trace dependency direction and existing callers before moving or extracting code. Do not introduce sideways feature imports, feature dependencies in primitives, private environment imports in browser-safe modules, or executable dependencies in reusable server code.
7. Keep types, tests, helpers, fixtures, and assets with their narrowest owner. Promote them only when multiple owners genuinely share the same meaning.
8. Keep routes and entrypoints thin. Move reusable product behavior to feature services and reusable infrastructure to primitives.
9. For implementation work, follow the repository's required impact analysis, editing, formatting, type-checking, and test workflow. Preserve behavior unless redesign is explicitly requested.
10. Summarize the resulting ownership choices, boundary decisions, important dependency constraints, verification performed, and any intentional deviations from the guide.

## Applying the Reference

- Preserve the core rules: narrowest ownership, explicit runtime boundaries, inward dependencies, thin adapters, typed contracts, and colocated tests.
- Adapt aliases such as `$lib` to the target project's configured aliases without changing the ownership model.
- Adapt optional examples for databases, CSS, background processes, and deployment to the target project's actual stack. Do not introduce a tool solely because it appears in the guide.
- Do not create empty directories, barrels, type files, or helper modules merely to match an example tree.
- When reviewing rather than editing, report concrete violations and recommended destinations without changing files.
- When asked for a target structure, derive it from actual product capabilities and runtimes; do not copy the example feature names verbatim.

## Decision Checklist

Before finalizing a placement or change, answer:

1. Who is the narrowest owner?
2. Which runtime may execute or import it?
3. Is it reusable behavior, a public contract, a primitive, or an adapter?
4. Does every dependency point toward common contracts or lower-level primitives?
5. Is the new file or directory justified by a distinct responsibility?
6. Are public exports intentional and internal imports still visible?
7. Are framework, network, environment, database, and process boundaries kept thin and explicit?
8. Are tests and validation located at the narrowest stable boundary?

Use the full review checklist and detailed examples in the reference for nontrivial changes.
