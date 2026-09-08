# SvelteKit Project Structure Style Guide

This guide defines reusable coding and ownership conventions for SvelteKit 5 applications. Adapt
the examples to the project's product language, configured aliases, persistence layer, styling
system, deployment model, and local instructions.

## Contents

- [Purpose](#purpose)
- [Architectural vocabulary](#architectural-vocabulary)
- [Dependency direction](#dependency-direction)
- [Feature and UI ownership](#feature-boundaries)
- [Component, type, and helper ownership](#svelte-component-ownership)
- [Reference data and assets](#reference-data-configuration-and-static-assets)
- [Public APIs and reactive state](#public-apis-and-indexts)
- [Browser data access and workers](#browser-data-access)
- [Routes and API audiences](#route-responsibilities)
- [Standalone services and commands](#standalone-services-and-commands)
- [Internal shared services](#internal-shared-services)
- [External-service adapters](#external-service-adapters)
- [The three meanings of "service"](#the-three-meanings-of-service)
- [Validation and server primitives](#validation-and-network-boundaries)
- [Database schemas and migrations](#database-schemas-and-migrations)
- [Naming and testing](#naming-conventions)
- [Framework roots and generated artifacts](#sveltekit-framework-roots-and-global-styling)
- [Environment boundaries](#environment-boundaries)

## Purpose

An application should remain understandable as its features, screens, and server operations grow. Code
should be organized by responsibility and feature ownership so that a developer can answer these
questions from a file's location:

- Which product capability owns this code?
- Can this code run in the browser, on the server, or in both environments?
- Is this a public contract, a feature implementation detail, or a low-level primitive?
- Where should related components, types, tests, and helpers be added?

The primary rule is: **place code with its narrowest true owner**. Promote code to a broader shared
location only after more than one owner genuinely needs it.

## Architectural vocabulary

Reusable application code includes feature implementations, common contracts, primitives, shared
application services, and integration adapters. HTTP routes, long-running processes, and one-shot
commands are executable adapters around that reusable code.

### Feature code

`$lib/feature/<feature-name>/` contains browser-safe implementation for a product capability, for
example `$lib/feature/account-settings/`.

Feature code may contain:

- Svelte components and component families
- browser-side state and interaction logic
- feature-specific formatting and view-model transformations
- browser API clients
- feature tests

Feature code must not import server-only modules, private environment variables, database code, or
filesystem/storage implementations.

### Server feature code

`$lib/server/feature/<feature-name>/` contains trusted server-side implementation for the same
product capability, for example `$lib/server/feature/account-settings/`.

Server feature code may contain:

- application services and use cases
- repositories and database queries
- authorization enforcement
- storage operations
- server-side transformations
- server-only tests

Server feature modules must not import Svelte components or browser implementation from
`$lib/feature/`. A server feature may import its matching common contracts and low-level server
primitives.

### Common feature code

`$lib/common/feature/<feature-name>/` contains code shared by the browser and server for one product
capability, for example `$lib/common/feature/account-settings/`.

Common feature code may contain:

- serializable domain and transport types
- schemas and validation shared across the network boundary
- constants and enums with identical browser/server meaning
- deterministic, side-effect-free transformations
- shared test fixtures that contain no secrets

Common code must be safe to include in a browser bundle. It must not import:

- `$lib/server/**`
- `$env/static/private` or `$env/dynamic/private`
- Node-only modules
- database clients or schemas that pull server dependencies into the browser
- Svelte components or browser state

`common/` is not a general utility directory. Product contracts belong under `common/feature/`.
Infrastructure-neutral common primitives may live directly under a precisely named common package,
but only when their meaning is genuinely application-wide.

### Primitives

Primitives are foundational capabilities that features build upon. They are organized by technical
responsibility rather than product workflow.

Examples include:

- `$lib/auth/` for browser-safe authentication identity and permission concepts
- `$lib/server/auth/` for credential verification, authorization enforcement, and server sessions
- `$lib/server/db/` for database connections and low-level database infrastructure
- `$lib/server/storage/` for object-storage infrastructure
- `$lib/ui/` for generic reusable visual and interaction primitives

Primitive modules must not depend on product features. A database connection module, for example,
must not import account-settings behavior.

A primitive may have a matching browser-safe contract package under `common/` when multiple
runtimes genuinely share its language. For example, `common/db/types.ts` may define a deliberately
public, serializable database-infrastructure contract used outside the server database package.
Database connections, transactions, query builders, table definitions, repository rows, and other
implementation details remain in `server/db/`.

Do not place a feature DTO in `common/db/` merely because it is persisted. An account update remains
owned by `common/feature/account-settings/`; storage is an implementation detail of that feature.

### Shared application capabilities

An internal shared service owns cohesive application behavior behind typed public
contracts, and several product features consume it. Examples include media/asset management with
a reference registry and delivery resolution, notification scheduling, document numbering, and
file conversion.

Place it under a dedicated server directory such as
`$lib/server/services/<service-name>/`. It may own SQL data, object-storage manifests, or no durable
state at all. Its cohesive application contract distinguishes it from a small utility such as a
URL formatter — a formatter remains a helper owned by its caller, not a service.

A shared service must not import product features. Feature-specific behavior crosses the
boundary as a named provider contract the service defines and features implement; the
application composition root registers implementations. A feature may also access primitives
directly for its own data — the shared-service layer exists where responsibilities are genuinely
shared, not as a mandatory wrapper around every query.

### Integration capabilities

An external-service adapter wraps an independently operated capability — payment gateway, email
delivery, PDF rendering — behind a local typed contract. Place adapters under
`$lib/server/integrations/<provider-name>/`.

The adapter owns protocol details, request signing, response/evidence verification, error and
timeout mapping, and credential injection. The consuming feature owns business state: orders,
amounts, retries, durable message or document intent, and idempotency of business effects.
Adapters are consumed through dependency injection at the composition root; they are never a
mandatory layer between features and their own logic, and they never write application tables.

## Dependency direction

Dependencies should flow inward toward common contracts and primitives:

```text
routes and API handlers
       |
       +--> feature/<name> ----------------> common/feature/<name>
       |          |
       |          +--> ui and browser-safe primitives
       |
       +--> server/feature/<name> ---------> common/feature/<name>
                  |
                  +--> server/services/<service-name> --> server primitives
                  +--> server/integrations/<provider> --> external provider
                  +--> server primitives
```

Arrows run from consumer to provider: a feature consumes a shared service; the shared service
coordinates primitives; adapters are injected by the composition root. Nothing points back down
at `server/feature/<name>` from a shared service or integration.

Allowed dependencies:

| From                                                 | May depend on                                                                                     |
| ---------------------------------------------------- | ------------------------------------------------------------------------------------------------- |
| `feature/<name>`                                     | matching `common/feature/<name>`, generic UI, browser-safe primitives                             |
| `server/feature/<name>`                              | matching common feature contracts, common primitives, server primitives, and explicit server APIs |
| `common/feature/<name>`                              | other browser-safe common code only when ownership is clear                                       |
| `server/services/<service-name>`                     | primitives, service-owned/common contracts, lower-level service APIs, injected integration contracts; never product features |
| `server/integrations/<provider-name>`                | server primitives and typed configuration; never product features or application tables             |
| `src/services/<name>` and `src/commands/<name>`       | server feature APIs, shared application services, integrations, server primitives, and common contracts |
| `ui`                                                | generic UI and browser-safe primitives; never product features                                     |
| primitives                                          | lower-level primitives; never product features or shared application services                       |
| route `.svelte` and universal load code              | feature code, generic UI, and browser-safe common code                                            |
| `+page.server.ts`, `+layout.server.ts`, `+server.ts` | owned server feature or shared-service APIs and common contracts                                    |

Avoid sideways feature dependencies. If feature A needs behavior from feature B, expose a small
public API from B or move the truly shared contract to a clearly owned common module. Do not create
a broad `shared.ts` to bypass the ownership decision.

Shared services may consume lower-level shared-service contracts or injected integration contracts
when the dependency has a clear purpose and remains acyclic. Their implementations never import
product features or the application composition root. Provider registration happens outside both
implementations; a provider callback must not re-enter the operation that invoked it.

## Feature boundaries

A feature represents a product capability or cohesive workflow, not a technical file type.

Good feature names describe user or system behavior:

- `account-settings`
- `checkout`
- `content-publishing`
- `notification-preferences`
- `reporting`

Avoid features named after vague technical categories such as `helpers`, `models`, `data`, or
`misc`.

Browser, server, and common feature folders with the same feature name describe different runtime
sides of one capability. They do not need identical internal structures.

For parent concepts containing distinct child capabilities, see [Feature families](feature-families.md).
It covers nesting, child public APIs, sibling dependencies, and family composition without turning
the parent into a universal repository or barrel.

### Placement example

The following is an ownership example, not the proposed final application tree:

```text
lib/
├── feature/
│   └── account-settings/
│       ├── AccountSettingsForm.svelte
│       ├── formState.ts
│       └── index.ts
├── common/
│   └── feature/
│       └── account-settings/
│           ├── types.ts
│           └── validation.ts
└── server/
    ├── feature/
    │   └── account-settings/
    │       ├── service.ts
    │       └── repository.ts
    ├── auth/
    └── db/
```

In this example, the form and its interaction behavior are browser feature code. Serializable update
contracts and validation shared by the UI and API are common code. Authorization, persistence, and
trusted workflow behavior are server feature code. If `formState.ts` is used only by the modal, it
should instead be colocated in a component package rather than treated as feature-wide code.

### Server structure example with shared services and integrations

The following ownership example shows where internal shared services and external-service
adapters sit relative to features and primitives. It is an ownership example, not a proposed
final application tree:

```text
src/lib/server/
├── feature/
│   ├── sku-catalog/
│   │   ├── service.ts
│   │   ├── repository.ts
│   │   ├── mediaProviders.ts        implements the shared service's provider contract
│   │   └── runtime.ts
│   └── checkout/
│       ├── service.ts
│       ├── repository.ts
│       └── runtime.ts
├── services/                        internal shared application capabilities
│   └── media-manager/
│       ├── contracts.ts             public contracts: references, delivery, administration
│       ├── manager.ts               implementation behind the contracts
│       ├── providers.ts             provider contract for feature-owned usage reports
│       └── runtime.ts               base composition without feature providers
├── integrations/                    external-service adapters (no business state)
│   ├── payment-gateway/
│   │   ├── contract.ts              application-facing inputs, results, typed errors
│   │   ├── adapter.ts               protocol, signing, evidence verification
│   │   └── config.ts                credential and environment boundary
│   ├── email-delivery/
│   │   ├── contract.ts
│   │   └── adapter.ts
│   └── pdf-rendering/
│       ├── contract.ts
│       └── adapter.ts
├── application/                     composition root
│   └── mediaRuntime.ts              registers feature providers and adapters
├── db/                              primitive: connections, transactions, schema
└── auth/                            primitive: identity and authorization mechanics
```

Reading of the example: `checkout` consumes the media-manager contracts and the payment adapter
through its runtime; the media manager never imports `feature/checkout`; the payment adapter
holds no order or amount state. `feature/sku-catalog/mediaProviders.ts` implements the provider
contract the media service defines, and `application/` registers it — preserving the layer
without a circular import.

## Generic UI ownership

`$lib/ui/` contains reusable visual primitives and interaction systems that do not belong to a
product feature. It may contain both small leaf components and larger component suites.

Simple icons are leaf UI components and should remain flat under `$lib/ui/icons/`:

```text
ui/
└── icons/
    ├── ChevronLeft.svelte
    ├── Play.svelte
    └── Trash.svelte
```

Do not create a directory, `types.ts`, or `index.ts` for each individual icon. A single
`ui/icons/index.ts` may define the collection's public exports. Keep an icon in one `.svelte` file
unless it develops real supporting behavior or files.

A component belongs in generic UI when it:

- has no knowledge of accounts, orders, articles, reports, or another product domain
- accepts generic typed inputs instead of feature DTOs or database rows
- reports generic interaction events instead of performing feature-specific mutations
- does not call feature API endpoints or import server modules
- can be rendered and understood without constructing a product feature

Being reused by two screens does not automatically make a component generic UI. If it implements a
product rule, it remains in the owning feature. A feature should adapt its domain data into a generic
UI contract and handle persistence outside the UI package.

Large reusable UI systems may drill ownership down through nested component packages:

```text
ui/
└── component-suite/
    ├── index.ts
    ├── types.ts
    ├── SuiteRoot.svelte
    ├── primary-view/
    │   ├── PrimaryView.svelte
    │   ├── types.ts
    │   └── rendering.ts
    └── controls/
        ├── Controls.svelte
        └── types.ts
```

In this example, suite-wide public contracts belong in the top-level `types.ts`. Contracts used only
by `primary-view` belong in its nested `types.ts`, and a type private to one Svelte component remains
inside that component. The suite's `index.ts` exposes only its intended public surface.

## Svelte component ownership

### Keep small leaf components simple

A small component may remain a single `.svelte` file. Do not create empty `index.ts`, `types.ts`, or
helper files merely to satisfy a directory template.

Flat collections are appropriate for:

- design-system primitives
- icons
- small presentational components without supporting code

### Use a component directory for a component package

A component deserves its own directory when at least one of these is true:

- it has child components
- consumers use exported types or an imperative API
- it has dedicated pure logic or tests
- it owns several files that change together
- it represents a recognizable subsystem or reusable editor

A component package may look like:

```text
component-package/
├── ComponentRoot.svelte
├── ComponentControls.svelte
├── ComponentPanel.svelte
├── types.ts
├── stateTransitions.ts
├── stateTransitions.test.ts
└── index.ts
```

Files are optional and should exist only when they have a real responsibility.

### Component responsibilities

Components should primarily:

- receive explicit props
- derive display state
- render markup
- emit explicit events or call passed callbacks
- coordinate browser-only effects such as focus, canvas drawing, or media playback

Extract deterministic calculations and state transitions into typed `.ts` modules. Keep unavoidable
DOM side effects explicit by passing their DOM node, context, and input snapshot as arguments.

Route components should compose feature components and coordinate route-level state. They should
not accumulate unrelated editors, API clients, transformation logic, and rendering subsystems in a
single file.

File length is a review signal, not a hard limit. When a component becomes difficult to describe in
one sentence, or contains multiple independent state machines, split by responsibility rather than
by arbitrary line count.

## Type ownership

Types follow the same narrowest-owner rule as runtime code.

### Private component types

Keep a type inside a `.svelte` file when only that component uses it:

```svelte
<script lang="ts">
	interface Props {
		disabled?: boolean;
		onconfirm?: () => void;
	}

	interface DragSession {
		pointerId: number;
		startX: number;
	}
</script>
```

This includes private `Props`, render snapshots, DOM state, and local action results.

### Public component types

Place externally consumed component contracts in an adjacent `types.ts`:

- imperative component handles
- event payloads
- reusable prop value types
- objects that callers must construct

Consumers should import public types from the component package, not from the `.svelte` file.

### Sibling-shared types

Place types shared by a component family at the nearest common parent. A mode shared by a root
component, controls, and panel belongs to their parent component package rather than to one child.

### Browser/server contracts

Place serializable contracts used on both sides in `$lib/common/feature/<feature-name>/`. Examples
include request payloads, response DTOs, shared validation schemas, and stable domain values.

Do not put browser-only view state into common contracts simply because it resembles a server DTO.
Use an explicit transformation between transport data and the feature's view model when their
responsibilities differ.

### Server-only types

Keep repository rows, transaction contexts, trusted service inputs, and storage details in the
owning `$lib/server/feature/<feature-name>/`, `$lib/server/services/<service-name>/`, or server primitive.

### Avoid global type collections

Do not add unrelated declarations to broad files such as `types.ts` at the root of the application.
A file named `types.ts` is acceptable inside a clearly owned feature or component package because
the directory supplies its meaning.

## Shared code and helper naming

Avoid vague filenames such as:

- `common.ts`
- `helpers.ts`
- `utils.ts`
- `misc.ts`

Prefer names that describe the owned behavior:

- `geometry.ts`
- `interpolation.ts`
- `accountStatus.ts`
- `formatters.ts`
- `selectionState.ts`
- `apiClient.ts`

A module should have one cohesive reason to change. Pure logic should be colocated with its tests.

If code is used by only one component, keep it with that component. If sibling components use it,
move it to their parent package. If browser and server implementations use it with the same
semantics, consider the matching common feature. Reuse alone does not automatically make code a
global primitive.

## Reference data, configuration, and static assets

Do not group unrelated values under a broad `configs/` directory. Classify data by meaning and
runtime ownership.

### Domain reference data

Versioned code tables, mappings, and other product data with identical browser/server meaning
belong to the owning common feature:

```text
common/
└── feature/
    └── product-catalog/
        └── reference-data/
            ├── categoryCodes.json
            ├── colorMap.json
            ├── loaders.ts
            ├── types.ts
            └── validation.ts
```

Load and validate reference data through an owned module rather than importing raw JSON throughout
the application. Record the source or version when operational correctness depends on a particular
dataset.

### Presentation constants

Rendering defaults, visual mappings, and interaction constants belong to the feature or generic UI
package that interprets them. A visual value does not become common domain data merely because more
than one component uses it.

### Runtime configuration

Environment-derived configuration belongs at the server, framework, service, or command boundary
that reads it. Pass typed configuration inward. Do not mix runtime configuration with static domain
reference data.

### Public static assets

Files under `static/` are intentionally public and URL-addressable. Keep only assets the deployed
application should serve there. Test fixtures, sample payloads, generated job artifacts, and
private operational data must not live in `static/`.

Feature-owned bundled assets should live with their feature. Generic visual assets belong to their
UI package. Test fixtures belong beside the narrowest test suite that owns them.

## Public APIs and `index.ts`

Use `index.ts` as an intentional public boundary, not as a required file in every directory.

Good uses include:

- the public surface of a component family
- a feature's small browser API
- a UI primitive collection

Prefer explicit exports:

```ts
export { default as ComponentRoot } from './ComponentRoot.svelte';
export type { ComponentHandle, ComponentMode } from './types.ts';
```

Avoid long chains of `export *` and nested barrels. Internal files should generally use direct
relative imports so that dependencies remain visible and circular dependencies are less likely.

## Reactive state and functions

Use Svelte 5 runes and follow these rules:

- use Svelte 5 runes such as `$state`, `$derived`, `$effect`, and `$props()` for component state
- follow the project's declaration convention for `$derived` bindings
- keep state component-scoped and pass it through props by default
- introduce context or shared state modules only when state genuinely spans component ownership
  boundaries
- pass reactive dependencies explicitly to functions
- return state changes as plain typed values rather than mutating component-scoped state implicitly
- keep effect bodies small and move their calculations into pure functions

Prefer:

```ts
interface ItemFiltersSnapshot {
	search: string;
	processingState: string;
}

function buildItemQuery(filters: ItemFiltersSnapshot): URLSearchParams {
	const query = new URLSearchParams();
	if (filters.search) query.set('search', filters.search);
	if (filters.processingState) query.set('processingState', filters.processingState);
	return query;
}
```

Avoid functions that silently read or write unrelated component state.

## Browser data access

Browser data access belongs to the feature consuming the endpoint. Components should receive data
and callbacks rather than constructing URLs, interpreting HTTP failures, or duplicating response
parsing throughout the UI.

A browser feature may expose an `api.ts` or another responsibility-specific client module:

```text
feature/
└── product-catalog/
    ├── api.ts
    ├── api.test.ts
    └── product-list/
```

Browser API functions should:

- accept all request inputs explicitly
- accept a `fetch` implementation when they may run with SvelteKit-provided `fetch`
- construct and encode URLs in one place
- use common feature contracts for request and response DTOs
- normalize non-success responses and network failures into typed feature errors or results
- validate untrusted response data when runtime guarantees are required
- return plain data rather than mutating component state

Route and feature components coordinate loading state and apply returned data. Presentational UI
components must not call product API endpoints. A component package may call browser primitives
that are intrinsic to its generic interaction behavior, but not feature persistence APIs.

Server load functions may call server feature services directly. Do not route an internal server
operation through HTTP merely to reuse a browser API client.

## Browser workers

Browser Web Workers are browser feature adapters, not standalone server services. Place a worker
with the feature or generic UI package that owns its computation:

```text
feature/
└── analysis-feature/
    └── workers/
        ├── calculation.worker.ts
        ├── client.ts
        ├── types.ts
        ├── calculation.ts
        └── calculation.test.ts
```

Worker ownership follows these rules:

- worker message contracts belong beside the worker and its main-thread client
- contracts shared only between browser threads do not belong in common server/browser contracts
- deterministic algorithms belong in ordinary `.ts` modules testable without spawning a worker
- `.worker.ts` adapts `onmessage`, `postMessage`, cancellation, and worker errors
- the main-thread client owns worker construction, request correlation, and termination
- workers must not import server code, DOM-dependent components, or private environment modules
- a worker belongs under `ui/<package>/` only when both its contract and computation are product-neutral

Pass input snapshots into workers and return result snapshots. Do not use a worker as hidden global
state.

## Route responsibilities

SvelteKit route files are framework adapters.

### Pages and layouts

`+page.svelte` and `+layout.svelte` should:

- compose feature components
- connect route data to feature view models
- own navigation-specific behavior
- coordinate only state whose lifetime is the route or layout

Reusable product behavior belongs in `$lib/feature/<feature-name>/`.

### Server load functions and actions

`+page.server.ts` and `+layout.server.ts` should:

- parse framework inputs
- invoke server feature services
- convert expected failures into SvelteKit responses
- return serializable DTOs

They should not contain substantial query construction or application workflows.

### API handlers

`+server.ts` files should be thin transport adapters:

1. authenticate and authorize the request
2. validate path, query, and body inputs
3. call a server feature service
4. map the result or known errors to HTTP

Database and storage operations belong behind server feature or primitive APIs.

### API audiences

When an application serves callers with different trust levels, make the intended audience visible
in route organization or route metadata. Common audiences include browser clients, third-party
integrations, background services, administrators, and internal system calls. Do not create
audience trees that the application does not need.

Document or enforce each protected API route's intended audience and matching authentication policy.
Moving a route between trust boundaries is a security change, not a file-organization cleanup.

Handlers for different audiences may call the same server feature service, but they must not call
one another over internal HTTP merely to reuse behavior. Each handler independently performs its
transport validation and audience-specific authorization before invoking shared application logic.

Service APIs must preserve applicable idempotency, tenancy, resource-ownership, and concurrency
controls in addition to identity authentication. Internal credential endpoints should expose only
the minimum operations required for establishing an application principal.

Shared request and response contracts remain owned by the corresponding common feature. Do not
create a broad package that groups unrelated product contracts only by transport audience.

## Standalone services and commands

Standalone executables are adapters around reusable server behavior. They are not general-purpose
libraries and must not become alternate homes for application logic. Do not confuse these
processes with internal shared services (`$lib/server/services/`) or external-service adapters
(`$lib/server/integrations/`); see [The three meanings of "service"](#the-three-meanings-of-service).

### Executable categories

If the repository includes standalone processes, use a clearly configured location such as
`src/services/<service-name>/` for long-running processes that own a runtime lifecycle, such as
polling, scheduling, signal handling, or graceful shutdown.

Use a corresponding location such as `src/commands/<command-name>/` for one-shot operational tasks
that perform an operation and exit, such as bootstrapping identities, applying an operational
binding, synchronizing data, or cleaning stale resources.

Tool-owned configuration entrypoints should live with the command or operation that owns them when
the tool requires a particular exported shape. They should remain narrowly focused on adapting the
tool to reusable server primitives.

When these executable locations fit the repository, use kebab-case directories and a small
`main.ts` entrypoint:

```text
src/
├── services/
│   └── notification-dispatcher/
│       ├── main.ts
│       ├── runtime.ts
│       └── runtime.test.ts
└── commands/
    └── data-import/
        ├── main.ts
        ├── options.ts
        └── options.test.ts
```

This example establishes the executable categories and file responsibilities. It does not prescribe
the final names or decomposition of existing processes.

### Thin entrypoints

`main.ts` is a composition root. It may:

- parse command-line arguments and environment variables
- validate executable configuration
- construct database, authentication, storage, and external-service clients
- register process signals and lifecycle handlers
- invoke a service runtime or command operation
- log startup, shutdown, and final outcomes
- set `process.exitCode` for a failed one-shot command

Move testable application behavior out of `main.ts`. Product operations shared with HTTP handlers
or other executables belong in `$lib/server/feature/<feature-name>/`. Reusable infrastructure belongs
in the appropriate `$lib/server/` primitive.

Executable code may contain adapter-specific orchestration, such as a polling loop or command-line
option parser, when that behavior has no meaning outside the process. Extract it into a focused
module beside `main.ts` once it needs independent tests.

### Dependency direction

Executable dependencies flow toward reusable server code:

```text
service or command main.ts
            |
            +--> executable runtime or options
            +--> server/feature/<name>
            +--> server primitives
                         |
                         +--> common contracts when required
```

Reusable server modules must never import from `src/services/` or `src/commands/`. This restriction
does not prohibit importing reusable `$lib/server/services/` contracts. One executable
must not import another executable's `main.ts`. If two entrypoints share behavior, place that
behavior under the narrowest server feature, shared application service, or primitive.

### Configuration and environment

Standalone entrypoints may use `process.env` because they run outside the SvelteKit module graph.
Read environment variables at the executable boundary, validate them once, and pass typed
configuration into runtime and service functions.

Do not let reusable server behavior reach into `process.env` implicitly. Do not copy generic
environment-file parsing into multiple entrypoints; use a deliberate executable configuration
primitive if several processes require identical behavior.

Secrets must remain ephemeral. Never log secret values or write bootstrap credentials to tracked
configuration files.

### Resource and lifecycle ownership

The executable that creates a resource owns its cleanup. This includes:

- database pools and transactions opened for the process lifetime
- storage and external-service clients
- timers and polling loops
- signal handlers
- temporary files and credential artifacts

Long-running services should support graceful `SIGTERM` and `SIGINT` handling, stop accepting new
work, finish or safely interrupt the active unit of work, and close owned resources.

Reusable services must not call `process.exit()`. One-shot entrypoints should prefer setting
`process.exitCode` after cleanup so `finally` blocks can run.

### Long-running service behavior

Keep one iteration of a polling or reconciliation cycle independently callable. The outer runtime
owns repetition, delay, leadership, cancellation, and logging; the cycle owns one unit of
application work.

Make time, polling intervals, cancellation state, and external dependencies explicit when doing so
allows deterministic tests. Avoid hidden module-level state except where the executable lifecycle
itself clearly owns it.

### Command behavior

Commands should be idempotent when operationally practical. Separate option parsing from the
operation so invalid input fails before resources are mutated.

A command should report what it changed without printing secrets. Expected operational failures
should produce a concise error and non-zero exit code; unexpected errors should retain enough
context for diagnosis.

### Testing executables

- unit test option and environment parsing as pure input-to-output behavior
- unit test polling cycles and command operations with injected dependencies
- test resource cleanup and cancellation behavior where failure could leak work or connections
- use integration tests for real database, storage, authentication, and leader-election boundaries
- keep `main.ts` small enough that most behavior is verified without spawning a process
- add a process-level smoke test only when startup wiring itself presents meaningful risk

## Internal shared services

Use `$lib/server/services/<service-name>/` for a cohesive in-process application capability with
an independent contract consumed by several features. It need not own a database table, registry,
network endpoint, or process. Reuse by several callers alone does not justify extracting policy
from its true product owner. Keep one directory per real service; the example files are optional:

```text
server/services/media-manager/
├── contracts.ts        public contracts: references, delivery, administration
├── manager.ts          implementation behind the contracts
├── providers.ts        provider contract for feature-owned behavior
├── runtime.ts          base composition without feature providers
└── runtime.test.ts
```

Rules:

- the service owns its internal state (if any) and coordination rules; it does not own the business
  relationships of its consumers (which SKU uses a media reference stays with the SKU feature)
- expose a small typed function API; `contracts.ts` and an explicit `index.ts` are optional ways to
  express it, not mandatory files. Routes retain audience authentication; owning features retain
  business authorization and lifecycle policy, while the service enforces its own scope invariants
- the service must not import from `$lib/server/feature/`; when feature-specific behavior is
  actually needed, accept a named provider contract implemented by the feature. Simple services
  can accept typed values and primitive clients without introducing a provider registry
- the application composition root (for example `$lib/server/application/`) constructs the
  service runtime and registers feature providers; a base runtime without providers keeps the
  service importable from contexts that must not depend on product features
- features may still access primitives directly for their own data; the layer exists only where
  responsibilities are genuinely shared
- promote a shared service out of a feature only when a second real consumer exists or the state
  is genuinely shared; do not pre-create service directories speculatively

### Project-artifact example

A project-artifact service consumed by intake, review, workflow execution, and purge can live in
`$lib/server/services/project-artifacts/` even when it owns no SQL tables. It owns canonical
project/inspection paths, manifests, media slots, file validation, signed URLs, and scoped transfers.
The object-storage primitive owns provider mechanics. Project/intake/trash features own lifecycle,
upload eligibility, and permission to delete; workflow owns stage dispatch and completion policy.
An ID in a storage path does not make the service the owner of that entity's lifecycle.

Pass explicit project and inspection scope into artifact operations. Do not infer a parent ID from
an inspection ID, select an arbitrary first child, or broaden inspection deletion to a project
prefix. Keep validators with artifacts but job-stage decisions with workflow; moving the directory
alone does not establish that boundary.

Keep reusable factories free of SvelteKit environment imports and singleton construction.
SvelteKit runtime modules bind application clients; standalone processes construct the same
factories with their own clients and typed configuration. A shared service does not need `main.ts`,
an HTTP hop, or another deployment. Schema definitions stay under `server/db/schema`; any SQL
queries for service-owned data belong in its repository, following [feature data access](feature-data-access.md).

SQL rollback cannot roll back object-storage writes. When a feature combines both, document who
owns retry, staging cleanup, and compensation; do not describe the combined operation as atomic.
Test the public capability with injected storage and configuration, including sibling-scope
isolation and partial failures. Test real database/storage boundaries separately when needed.

## External-service adapters

Create `$lib/server/integrations/<provider-name>/` to wrap an independently operated provider.
Keep the application-facing contract separate from the transport implementation:

```text
server/integrations/payment-gateway/
├── contract.ts         application-facing inputs, results, and typed errors
├── adapter.ts          protocol, signing, evidence verification, error and timeout mapping
├── config.ts           credential and environment boundary
└── adapter.test.ts     protocol fixtures; no real provider calls
```

Rules:

- the adapter owns protocol details, signing, response and evidence verification, error and
  timeout classification, and credential injection; it never owns business state
- orders, amounts, payment attempts, retries, durable message or document intent, and business
  idempotency stay in the consuming server feature
- the contract states when a capability does not exist (no query API, no cache invalidation)
  rather than letting callers assume it
- adapters are injected as dependencies by the composition root; features depend on the typed
  contract, never on the adapter module or the provider's SDK types
- adapters never import application tables or feature modules, and never write application state
- test adapters against recorded protocol fixtures; routine tests must not call real providers

## The three meanings of "service"

This guide uses "service" in three distinct senses. Classify explicitly rather than letting the
word choose the directory:

| Term | Meaning | Directory | Owns |
| --- | --- | --- | --- |
| Internal shared service | Cohesive in-process application behavior consumed by several features | `$lib/server/services/<name>/` | Its contracts, coordination rules, and any internal state |
| External-service adapter | Local typed wrapper around an independently operated provider | `$lib/server/integrations/<name>/` | Protocol, signing, verification, credentials — no business state |
| Standalone service executable | A long-running process with its own runtime lifecycle | `src/services/<name>/` | The process: polling, signals, graceful shutdown |

A standalone executable may compose shared services and adapters, but shared services and
adapters never import executables. When documenting architecture (for example a feature-layout
directory), record ownership separately from deployment. A self-hosted model server is a deployed
process; its SvelteKit client is an integration adapter. Calling that deployment an internal service
does not place its implementation in `$lib/server/services/`. Feature-local `service.ts` is also a
valid application-service implementation and does not become shared merely because of its name.

## Validation and network boundaries

Treat all network and form input as untrusted.

- validate at the boundary before invoking application behavior
- colocate a browser/server-shared schema with its common feature contract
- keep server-only validation near its server service
- derive TypeScript types from schemas when doing so preserves a clear domain model
- distinguish transport DTOs from internal database rows and UI view models
- use explicit mapping functions between those representations

Do not expose database row shapes as accidental API contracts.

## Authentication, database, and storage primitives

Technical primitives should expose small, stable interfaces to server features.

- `$lib/auth/` contains browser-safe identity, permission, and routing concepts
- `$lib/server/auth/` contains trusted authentication and authorization mechanisms
- `$lib/server/db/` owns connections, transactions, and database infrastructure
- `$lib/server/storage/` owns object-storage infrastructure

Feature-specific authorization decisions still belong with the server feature when they express a
product rule. The auth primitive should provide the mechanism used to enforce that decision.

Features should not import a database connection merely for convenience when an owned repository
or service boundary would make the behavior clearer and testable.

For implementation rules, composition patterns, exceptions, and a migration checklist, read
[Feature data access](feature-data-access.md).

## Database schemas and migrations

Database table definitions, relations, inferred row types, and query infrastructure are server-only
database implementation. Keep them under `$lib/server/db/`:

```text
server/
└── db/
    ├── connection.ts
    ├── transactions.ts
    └── schema/
        ├── accounts.ts
        ├── products.ts
        ├── orders.ts
        ├── relations.ts
        └── index.ts
```

The exact schema split should follow cohesive database ownership and relation requirements rather
than forcing one file per feature. Feature repositories import tables through the server database
schema boundary.

Rules:

- ORM table definitions and inferred persistence rows never belong in browser-safe common code
- database row types are not API response types or feature view models
- map rows to explicit common DTOs or server feature models at a repository or service boundary
- server features own product queries; `server/db` owns connections, transactions, schema, and
  genuinely cross-feature database mechanics
- feature repository modules are the normal consumers of ORM schema objects; routes and services
  consume feature APIs rather than constructing queries
- review schema changes and their generated migration together

Keep migration output and ORM metadata in the locations required by the selected database tool.
Review generated SQL and metadata with the source change that produced them; do not manually
reorganize tool-managed files unless the tool explicitly supports it. Keep ORM configuration as a
narrow composition root.

Do not edit old applied migrations to represent a new schema change. Add a new migration through the
repository's migration workflow.

## Naming conventions

- use kebab-case for feature and package directories: `account-settings`
- use PascalCase for Svelte components: `AccountSettingsForm.svelte`
- use camelCase for TypeScript implementation modules: `selectionState.ts`
- use `.test.ts` next to the pure module being tested
- follow the project's configured TypeScript import-extension convention consistently
- use kebab-case for directories under `services/` and `commands/`
- name standalone executable entrypoints `main.ts`
- choose domain-specific names over abbreviations unless the abbreviation is established product
  language
- name callbacks after the event or intent, following the project's established casing convention

Avoid version prefixes such as `V2` in new feature names unless both versions are actively supported
as separate concepts. Migration state should not become permanent domain language.

## Testing conventions

- test deterministic feature behavior in colocated `.test.ts` files
- test state transitions without rendering a component when they can be represented as pure input
  and output
- test server feature services through their public interface
- use integration tests for database repository behavior and route boundaries
- keep fixtures owned by the narrowest feature that uses them
- place browser/server-safe shared fixtures in `common/feature/<feature-name>/` only when both sides
  genuinely use them

Component tests should focus on observable interaction and rendering behavior rather than private
implementation details.

For automated checks of runtime zones, owner boundaries, public exports, and cycles, see
[Boundary enforcement](boundary-enforcement.md). Validate forbidden as well as allowed edges;
typechecking alone does not enforce architectural ownership.

## SvelteKit framework roots and global styling

SvelteKit root files are framework composition points with narrow responsibilities:

- `hooks.server.ts` composes request-wide server primitives such as authentication, audience
  enforcement, and request context; reusable mechanisms remain in `$lib/server/`
- `app.d.ts` contains ambient SvelteKit application declarations only
- `app.html` contains the global HTML document shell only
- the configured global stylesheet owns design tokens, resets, typography defaults, and
  intentionally global styles

Do not place product feature workflows in framework roots. If request-wide logic grows, extract it
into a server primitive and keep the hook responsible for ordering and composition.

Feature styles remain with feature components. Generic UI consumes global design tokens but must
not add product-domain rules to the global stylesheet. Reusable Svelte actions and browser behaviors such as
focus trapping belong under an owning generic UI package, for example `$lib/ui/actions/`.

Generated SvelteKit types such as route `$types` are framework output. Import them from their route
but do not move, wrap, or manually edit them.

## Review checklist

Before adding or moving code, ask:

1. Which feature or primitive owns this behavior?
2. Does it run in the browser, on the server, or identically in both?
3. Is this type private, component-public, sibling-shared, cross-boundary, or server-only?
4. Is a new file clarifying a responsibility, or only satisfying a template?
5. Does this import follow the allowed dependency direction?
6. Can the logic be a pure function with explicit inputs and returned state changes?
7. Is the route acting as an adapter, or accumulating application behavior?
8. Does an `index.ts` define a useful public API, or hide internal dependencies?
9. Is a generic filename becoming a dumping ground?
10. Is the behavior tested at its narrowest stable boundary?
11. Is an executable correctly classified as a long-running service or a one-shot command?
12. Is `main.ts` limited to configuration, dependency construction, lifecycle, and outcome handling?
13. Is browser network access owned by a typed feature client rather than a presentational component?
14. Is browser-worker computation separated from its messaging adapter and testable as pure logic?
15. Is a dataset classified as domain reference data, presentation configuration, runtime
    configuration, a public asset, or a test fixture?
16. Are database tables and row types confined to server database and repository boundaries?
17. Is a SvelteKit framework root limited to framework composition?
18. Does every API handler retain an explicit caller audience and its required authorization
    fencing?
19. Does a shared application capability have an independent contract and multiple real consumers,
    rather than merely a reused product operation or generic helper? Persistent state is optional.
20. Does the shared service define provider contracts instead of importing product features, with
    implementations registered at the composition root?
21. Does the external-service adapter own only protocol, verification, and credentials, with
    business state remaining in the consuming feature?
22. Is the word "service" used for exactly one of the three meanings, with the directory matching
    the meaning?

## Generated, build, and local artifacts

Generated and machine-local artifacts are not architectural source packages:

- `.svelte-kit/` and the configured adapter output directory are generated build output
- migration metadata is generated through the database migration workflow
- local credential material belongs only in its gitignored, permission-restricted runtime location
- generated job output and user-uploaded media remain outside source directories

Do not import application source from build output. Do not commit secrets, local credentials,
generated media, or machine-specific paths. Tool-generated files that are intentionally versioned,
such as migrations and lockfiles, should be changed only through their owning tool or package
workflow and reviewed with the source change that produced them.

## Environment boundaries

- server-only SvelteKit modules use private environment modules; prefer `$env/static/private` when
  values are available at build time and `$env/dynamic/private` when runtime lookup is required
- browser-safe public values use the corresponding public environment module and must not contain
  secrets
- use SvelteKit environment modules rather than `process.env` inside the SvelteKit module graph
- standalone services and commands outside the SvelteKit module graph may use `process.env`
- never import a private environment module through `common/` or `feature/`

Every newly required environment variable must be documented and provided to build and deployment
environments.

## Adoption

Apply this guide incrementally:

- all new code should follow it
- touched code should move toward it when the change remains reviewable
- avoid repository-wide mechanical moves without a feature-by-feature plan
- establish and review a target directory structure before beginning the migration
- preserve behavior while moving ownership boundaries; functional redesign should be a separate,
  explicit change

When existing code conflicts with the guide, treat the conflict as migration work. First account for
local constraints and established public contracts, then improve the structure in reviewable steps.
