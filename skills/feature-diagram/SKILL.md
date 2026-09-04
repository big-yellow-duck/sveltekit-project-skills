---
name: feature-diagram
description: Create or update feature dependency diagrams for a codebase using Mermaid composition and dependency semantics. Use when mapping features and subfeatures, documenting which features compose which subfeatures versus which features depend on which contracts, drawing primitives as a foundation band instead of node spaghetti, distinguishing sidecar subfeatures consumed through a parent runtime, maintaining a rework or roadmap table beside the diagram, or deciding whether a capability is a feature, a subfeature, or a primitive.
---

# Feature Diagram

Draw feature architecture as a single Mermaid flowchart that separates two questions readers
always ask: *what is this made of?* and *what does this consume?* Enclosure answers the first,
arrows answer the second. Never mix the two in one arrow.

## Workflow

1. Inventory the capabilities in scope: ship-ready features, features under rework, and queued
   features. Ask for the list if the repository does not document it.
2. Classify each capability as a **feature**, a **subfeature** (composed inside a feature), a
   **sidecar** (composed and consumed from outside through the parent's runtime), or a
   **primitive** (a building block: database, object storage, identity provider, message bus).
   Read [references/diagram-guide.md](references/diagram-guide.md) completely before drawing.
3. Place subfeatures as compartments inside their parent feature box. If a capability has no
   independent client surface or lifecycle, it is a compartment, not a node.
4. Draw solid arrows only for runtime dependencies between top-level features and between a
   feature and another feature's named subfeature. Draw dashed arrows for sidecar consumption.
5. Collapse primitives into a foundation band with no inbound edges, and tag every feature node
   with the primitives it uses using the `⚑` marker.
6. Color nodes by roadmap state (`complete`, rework, next target, queued) with one `classDef` per
   state, and keep a rework table under the diagram mapping every colored node to the concrete
   change required.
7. Verify the diagram compiles with mermaid-cli (`mmdc`) before delivering it, and re-check after
   every edit. A diagram that does not compile is not a deliverable.
8. Report the composition decisions (what became a compartment and why), the dependency edges you
   drew or deliberately removed, and any capability you left out of scope.

## Reference routing

Read [references/diagram-guide.md](references/diagram-guide.md) for:

- The full vocabulary: feature, subfeature, sidecar, primitive, composition, dependency.
- Mermaid patterns for feature boxes with compartments, the primitives band, sidecar styling,
  and per-node primitive tags.
- Layout and dark-theme configuration that renders legibly.
- Compile verification with mermaid-cli and the failure modes to check.
- Worked examples: a two-level container model (project → inspections) with sidecar artifacts,
  distributed trash ownership, and folder placement as a prerequisite feature.

## Core constraints

- Composition is enclosure, never an arrow. Dependency is an arrow, never enclosure.
- A subfeature has no client surface or lifecycle of its own; if it does, it is a feature.
- Primitives have no inbound arrows. Their consumption is documented as a tag on the consumer.
- Every node states its primitive dependencies; every colored node appears in the rework table.
- Verify compilation with mermaid-cli in both default and dark themes before delivering.
- Adapt node names, edges, and tags to the target repository; do not copy the example domains.
