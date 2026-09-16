# SvelteKit feature families

Use a family when one product concept owns several cohesive capabilities with distinct behavior,
contracts, or tests. A family is an ownership boundary, not another runtime category. Apply the
normal browser, server, common, and dependency rules within it.

## Choosing the boundary

- Nest a capability when its meaning and lifecycle belong to the parent concept: project intake,
  project inspections, or project trash.
- Keep an independently meaningful capability separate even when the UI presents it beside the
  family. Folder management does not become project-owned merely because folders contain projects.
- Extract an internal shared service only for an independently owned application capability, not
  because two children call the same parent operation. Project artifacts can serve several
  features while project deletion eligibility stays with project trash.
- A small feature can remain flat. Do not invent children, mirrored runtime trees, or empty files
  just to match an example. Route nesting and table foreign keys inform but do not decide ownership.

For example, these server owners can coexist:

```text
lib/server/
├── feature/
│   ├── project/
│   │   ├── repository.ts       container identity and placement
│   │   ├── runtime.ts          parent-owned operations and composition
│   │   ├── intake/            upload draft lifecycle
│   │   ├── inspections/       inspection reads and metadata
│   │   └── trash/             project and inspection deletion policy
│   └── folders/
│       └── trash/             folder subtree policy
└── services/
    └── project-artifacts/     structured artifact access
```

Create corresponding browser or common child packages only where those runtimes need them.
Keep child types, fixtures, tests, and helpers with the child. Put family-wide contracts at the
parent only when several children genuinely share the same meaning. A child's transport DTO does
not become parent-owned because a parent page displays it.

## Public APIs and composition

Choose an explicit entrypoint for each capability that has outside consumers. An entrypoint can be
a narrow `runtime.ts`, an explicit `index.ts`, or named factory/contract exports; the filename alone
does not make every export public.

- The parent exposes parent behavior and any deliberately composed family operations. It need
  not forward every child method or become the mandatory import path for all consumers.
- A child may expose a direct public API such as `project/intake/runtime.ts`. Consumers should not
  traverse its private repository helpers, even if they are siblings in the same family.
- Siblings may consume one another's explicit APIs when the dependency is purposeful and acyclic.
  Do not duplicate an inspection read in intake just to avoid an owned inspection API.
- Child implementations must not import a parent runtime/barrel that constructs or re-exports
  that child. Import a lower-level family contract or receive the needed operation explicitly.
- Compose multi-child operations at the narrowest owner of the invariant. If composition would
  produce a cycle, inject named contracts at the parent or application runtime. Do not conceal
  sibling dependencies behind a broad family `shared.ts` or service locator.

Repositories keep their owned queries. A parent repository is not a universal repository for all
descendants. For a transaction spanning children or features, the initiating owner supplies the
transaction to their coordination contracts; nested methods must not silently use a singleton.
See [feature data access](feature-data-access.md) for transaction and persistence rules.

Folder trash, for example, can own the subtree transaction and consume project-trash coordination
methods. Its runtime constructs the folder API; project trash does not construct or export folder
operations simply because the transaction includes projects. Shared persistence does not dictate
which runtime owns that API.

## Checking a family

Check that parent operations, each child's public surface, and permitted sibling edges are clear.
Tests should exercise child behavior independently and family-level coordination where it exists.
For automated import and export checks, use [boundary enforcement](boundary-enforcement.md).
Directory nesting must not turn the entire family into an unrestricted import zone.
