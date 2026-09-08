# Boundary enforcement

Use this reference when establishing or changing architecture checks. Checks enforce the project's
agreed ownership rules; they do not decide product ownership or whether a migration is complete.
Do not install a new tool or rewrite CI for an ordinary component edit merely because this skill
is active. Prefer the repository's existing lint, graph, build, and test mechanisms.

## Define the policy before encoding it

Identify runtime zones, feature owners (including nested children), public entrypoints, permitted
cross-owner edges, and composition roots. Configure the deepest matching owner: recognizing only
`feature/project` would miss a private import from `project/intake` into `project/inspections`.
Use the [dependency rules](style-guide.md#dependency-direction) and the project's explicit choices.

The rules worth enforcing are:

| Rule | Check |
| --- | --- |
| Browser/server isolation | Browser and common entrypoints cannot reach server code, private configuration, or Node-only implementations, including through barrels |
| Ownership direction | Shared services/primitives cannot import product features; reusable code cannot import standalone executable modules |
| Public surfaces | Cross-owner imports and re-exports target declared entrypoints and exports, not arbitrary internal files |
| Feature families | Sibling edges follow declared child APIs; a child cannot depend on a parent runtime that composes it |
| Persistence access | ORM schemas are consumed by owning repositories and documented infrastructure; singleton clients are bound at composition roots |
| Cycles | Detect runtime cycles through relative imports, aliases, re-exports, and literal dynamic imports |

Do not classify every `index.ts` or `runtime.ts` as universally allowed. A runtime may be public to
SvelteKit routes but unsuitable for a standalone worker because it binds framework configuration.
Distinguish reusable factory/contract entrypoints from environment-bound composition entrypoints.

## Resolve the graph, not just import text

Use compiler-aware or parser-backed tooling that understands the project's TypeScript aliases,
relative paths, extension convention, and Svelte script blocks. Equivalent resolved paths must
receive the same classification. Inspect re-exports as edges so a barrel cannot hide a violation.

Treat type-only imports separately: they still cross ownership boundaries and must use public
contracts, but they are not runtime cycles. Do not claim browser safety merely because a source
string lacks `$lib/server`; check reachable runtime modules and retain framework/build checks.
Report unresolved imports and non-literal dynamic loading as coverage limitations rather than
silently treating them as safe. Document tool limitations for generated modules or Svelte syntax.

A text search is useful for discovery or a narrowly stated regression guard, not proof of a valid
dependency graph. Likewise, typechecking alone does not enforce feature ownership.

## Verify that the rules catch violations

Exercise the checker with small isolated fixtures containing allowed and forbidden edges. At
minimum, cover the rules being added, including legitimate exceptions. Useful cases include:

- Allow a route to consume an intake public API; reject a sibling's private inspection helper.
- Reject the same private target reached by a relative path, alias, or re-exported barrel.
- Allow a child to import a family contract; reject a cycle through its parent composition root.
- Allow a shared service to use storage; reject its import of a product repository.
- Allow an exported server-only contract between server owners; reject a browser runtime path
  reaching private configuration. Keep ownership failures distinct from runtime-safety failures.
- Allow an owning repository's schema import; reject a route constructing product queries.

Assert the offending source, resolved target, and rule in diagnostics, and a failing exit status
for violations. A passing run over valid source alone does not demonstrate that restrictions work.
Validate fixtures against the actual checker, not a separate regex approximation of its rules.

When enforcement is part of the requested change, expose a repeatable command through the existing
project tooling and integrate it with the agreed validation workflow. Keep generated/build output
out of source ownership checks, but do not blanket-exempt tests, aliases, or entire feature trees.
Tests may access their owner's internals; that is not permission for production consumers to do so.

## Exceptions and limits

Keep authorized exceptions specific to an edge, export, or narrowly identified infrastructure
operation, with a reason. Report existing violations separately from newly introduced ones; do not
silently expand an allowlist or remove compatibility APIs to make the checker pass. Decisions
about accepted exceptions and migration completion remain with the project owner.

Static checks cannot establish permission correctness, transaction atomicity, storage deletion
scope, or provider-callback behavior. Pair them with focused behavioral tests where those risks are
in scope. Report which rules ran, what they covered, and what remains unverified; a green import
check is not a general claim that all architectural boundaries are correct.
