# Feature data access

Use this reference when implementing or reviewing persistence inside a SvelteKit server feature.
Standardize ownership and dependency direction without turning the database primitive into a
collection of product-specific CRUD methods.

## Contents

- [Boundary model](#boundary-model)
- [Repository contract](#repository-contract)
- [Runtime composition](#runtime-composition)
- [Services and routes](#services-and-routes)
- [Transactions](#transactions)
- [Exceptions](#exceptions)
- [Testing](#testing)
- [Migration checklist](#migration-checklist)
- [Review checklist](#review-checklist)

## Boundary model

Separate database infrastructure from feature persistence:

```text
src/lib/server/
├── db/
│   ├── connection.ts
│   ├── transactions.ts
│   └── schema/
└── feature/
    └── project-management/
        ├── repository.ts
        ├── service.ts
        └── runtime.ts
```

Adapt names and nesting to the repository. Preserve these responsibilities:

- The database primitive owns connection creation, pooling, transaction mechanics, ORM setup,
  schema definitions, and genuinely cross-feature database infrastructure.
- Each server feature owns the queries and persistence operations that express its product rules.
- Repository modules import ORM tables and build queries.
- Runtime composition roots bind repositories to the application database singleton.
- Services and routes consume feature-owned methods instead of importing schemas or constructing
  queries.

Do not add feature CRUD methods to a generic database primitive. That reverses ownership and makes
the primitive depend conceptually on every product capability.

## Repository contract

Create repositories around cohesive feature behavior, not around tables. A repository may use
several tables to load or mutate one feature aggregate. Split it only when the resulting modules
have distinct product responsibilities and callers.

Use the repository factory or injected-client convention already established by the project. A
factory keeps construction explicit and supports both the application client and transactions:

```ts
import { eq } from 'drizzle-orm'
import type { Database } from '$lib/server/db/connection.ts'
import { projects } from '$lib/server/db/schema/projects.ts'

export function createProjectRepository(db: Database) {
  return {
    async findById(id: string): Promise<Project | null> {
      const [row] = await db.select().from(projects).where(eq(projects.id, id)).limit(1)
      return row ? mapProject(row) : null
    }
  }
}
```

An equivalent set of functions that accepts `db` as the first argument is valid when that is the
project convention. Prefer one convention within a feature.

Make repositories responsible for:

- query construction, joins, ordering, pagination, locks, and database-specific expressions
- persistence row mapping and normalization
- feature-owned multi-table mutations
- translating expected persistence outcomes into explicit feature results
- preserving database constraints and invariants close to the write

Return feature models or explicit DTOs. Do not leak inferred row types as accidental service, route,
or browser contracts. Raw SQL is acceptable inside a repository when it is the clearest or safest
way to express the operation; the boundary is about ownership, not ORM purity.

## Runtime composition

Keep access to the application database singleton in a narrow runtime module:

```ts
import { db } from '$lib/server/db/connection.ts'
import { createProjectRepository } from './repository.ts'

export const projectRepository = createProjectRepository(db)
```

Import this bound repository from SvelteKit routes and application services. Keep the unbound
factory available to tests, commands, workers, and other composition roots that own a different
client lifecycle.

Do not make standalone services or command-line programs import a SvelteKit runtime singleton when
they create and close their own database client. Construct the repository with that local client.

Avoid module-level database access in browser-reachable modules. Keep repository, runtime, and
schema imports under an unambiguous server-only boundary.

## Services and routes

Use a service when persistence participates in business rules, authorization decisions, object
storage, external APIs, or multiple feature operations. Let the service depend on a repository
interface or bound feature API; do not let it import ORM schema objects.

Keep routes focused on transport concerns:

- authenticate and authorize the request
- parse and validate input
- invoke a feature service or repository method
- map the result to the response contract
- translate known feature errors into transport errors

Do not construct ad hoc queries in `+page.server.ts`, `+server.ts`, hooks, or endpoints. If a query is
small, it still belongs to the feature repository because ownership matters more than line count.

During migration, preserve a stable feature API with a thin compatibility facade when changing all
callers at once would be risky. Remove the facade after callers converge on the owned boundary.

## Transactions

Place a transaction inside a repository operation when one feature invariant requires multiple
database statements. Pass the transaction client explicitly to internal helpers or construct a
repository against it.

When a service needs an atomic operation spanning several persistence steps, expose one
transaction-aware feature operation rather than chaining methods that each use the application
singleton. If multiple features must participate, place orchestration at the narrowest server owner
that understands the cross-feature invariant and inject transaction-capable repository contracts.

Never hide transaction state in mutable module globals. Make the client or transaction dependency
visible in construction or function arguments.

## Exceptions

Permit direct low-level database access only where the low-level mechanism is the actual
responsibility, for example:

- authentication adapters that require a driver pool
- migrations, schema bootstrap, health checks, and database maintenance
- advisory locks, LISTEN/NOTIFY, replication, or other database-native infrastructure
- application composition roots that construct feature repositories

Document exceptions near the import. Keep product queries out of these modules. An infrastructure
exception does not justify schema imports in unrelated routes or services.

## Testing

Test each layer at its contract:

- Run repository integration tests against an isolated database or transaction and assert mapping,
  constraints, ordering, and write behavior.
- Unit-test services with a small repository interface or fake when the service owns meaningful
  branching and orchestration.
- Test routes for validation, authorization, status codes, and response contracts without
  duplicating repository coverage.

Repository factories make dependencies replaceable without introducing a global service locator or
mocking the ORM throughout the application.

## Migration checklist

1. Inventory direct database-client, schema, ORM-table, and raw-SQL imports in the feature and its
   routes.
2. Classify legitimate infrastructure integrations separately from product queries.
3. Identify cohesive feature operations from actual callers; do not generate one repository method
   per table operation mechanically.
4. Create a feature-owned repository and move query construction, row mapping, and transaction logic
   into it without changing behavior.
5. Add a runtime composition module that binds the repository to the application database client.
6. Update services and routes to consume feature methods. Keep authorization and transport mapping
   in their existing owners.
7. Preserve stable exports with a temporary facade when needed, then migrate callers incrementally.
8. Search again for forbidden direct imports and run focused repository tests, route tests, type
   checks, and the repository's architecture checks.

## Review checklist

- Does every product query have one clear feature owner?
- Do only repository modules import feature tables under normal application code?
- Do only composition roots import the application database singleton?
- Are row types mapped before crossing feature or transport boundaries?
- Is transaction ownership explicit and aligned with the invariant?
- Are infrastructure exceptions narrow and documented?
- Can repositories be constructed with a test or standalone client?
- Do routes and services remain free of query construction?
