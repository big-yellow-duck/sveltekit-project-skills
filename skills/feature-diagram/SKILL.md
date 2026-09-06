---
name: feature-diagram
description: Document a codebase's feature architecture as a feature-layout directory, or update an existing one — an overview README with a system diagram, detail views per product feature and shared service, operation flows, external-service adapter contracts, and an implementation roadmap, with Mermaid diagrams that separate composition from dependency. Use when mapping features, subfeatures, shared services, and primitives; drawing feature dependency diagrams; promoting a sidecar into a shared service with its own view; writing per-feature implementation-alignment tables; or deciding whether a capability is a feature, a shared service, a subfeature, or a primitive.
---

# Feature layout

Document feature architecture as a small directory of Markdown views, not one oversized page:
an overview README with a system diagram, a detail view per product feature and shared service,
an operation-flow document per multi-step workflow, adapter contracts per external service, and
a roadmap. Every diagram separates two questions readers always ask — *what is this made of?*
(enclosure) from *what does this consume?* (labelled arrows). Never mix the two in one arrow.

## Workflow

1. Inventory the capabilities in scope: implemented features, drafted features, shared services,
   and queued work. Ask for the list when the repository does not document it, and read existing
   feature docs before writing new ones.
2. Classify each capability as a **feature**, a **shared service**, a **subfeature** (composed
   inside a feature), or a **primitive** (database, object storage, identity provider, message
   bus). A capability with persistent state and reusable contracts consumed by several features
   is a shared service, not a sidecar compartment. Read
   [references/diagram-guide.md](references/diagram-guide.md) completely before drawing.
3. Choose the documentation surface. A single diagram answers a focused question; a
   feature-layout directory is the default when features own workflows, contracts, and migration
   state. For a directory, follow the skeleton in
   [references/layout-conventions.md](references/layout-conventions.md) and give every feature
   and shared service its own detail view.
4. Draw per-view Mermaid diagrams with the dark-theme init, a primitives band with no inbound
   arrows, `⚑` tags on every consumer, and arrows running **from the provider to the consumer** —
   A → B means B depends on A.
5. Write detail views that separate target design from current implementation: an ownership
   statement, public-contract tables, dependencies-consumed tables, and an implementation-
   alignment table (current evidence vs remaining work). Label not-yet-existing APIs *proposed*;
   never present them as implemented.
6. Document multi-step workflows as operation flows (execution order, decisions, transaction
   boundaries) kept separate from structural views, and external providers as adapter contracts
   in the external-services directory.
7. Record implementation status in prose and roadmap tables ("Status: **planned — …**",
   checklists with unchecked boxes). Never color diagrams by roadmap state; colors are
   structural, not temporal.
8. Verify before delivering and after every edit: compile every diagram with mermaid-cli
   (`mmdc`) in default and dark themes, and check every relative Markdown link resolves. A
   diagram that does not compile or a link that 404s is not a deliverable.
9. Report the classification decisions (what became a shared service vs a subfeature and why),
   the contract edges drawn or deliberately removed, the proposed-vs-implemented boundary, and
   anything left out of scope.

## Reference routing

- [references/diagram-guide.md](references/diagram-guide.md) — diagram vocabulary, the
  classification questions, Mermaid skeletons, provider→consumer arrow orientation, the
  primitives band, sidecars and when to promote them to shared services, status markers, dark
  theme, compile verification, and a worked example.
- [references/layout-conventions.md](references/layout-conventions.md) — the feature-layout
  directory skeleton, detail-view anatomy, prose conventions, operation flows, external-service
  adapter docs, roadmap format, and link hygiene.

## Core constraints

- Composition is enclosure, never an arrow. Dependency is an arrow, never enclosure.
- Arrow orientation is semantics: **A → B means B depends on A**; the arrow starts at the
  provider of the contract.
- A shared service is a documentation unit of its own (own view under `services/`), not a
  compartment of its main consumer and not a dashed sidecar.
- Primitives have no inbound arrows; their consumption is documented as a `⚑` tag on the
  consumer.
- Diagram colors never encode roadmap status; status lives in prose, alignment tables, and the
  roadmap.
- Detail views separate intended design from implementation gaps; unimplemented APIs are
  labelled *proposed*.
- Verify mmdc compilation in both themes and relative-link resolution before delivering.
- Adapt names, contracts, and examples to the target repository; do not copy the worked-example
  domains.
