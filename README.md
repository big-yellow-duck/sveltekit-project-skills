# SvelteKit project skills

A reusable collection of focused Codex skills for keeping SvelteKit projects consistent without forcing every task through one oversized prompt.

## Skills

| Skill | Use it for |
| --- | --- |
| `sveltekit-best-practices` | Everyday Svelte 5 and SvelteKit implementation plus project structure, state ownership, runes, SSR, accessibility, routes, server code, databases, services, and dependency direction |
| `feature-diagram` | Mermaid feature architecture diagrams: composition (feature boxes with subfeature compartments), runtime dependency arrows, sidecar subfeatures, primitives as a foundation band, roadmap coloring, and a rework table |
| `portable-svelte-components` | Figma/design-to-code work and reusable, data-driven component families |

The skills are intentionally composable. A design-to-code task may combine portable components with SvelteKit best practices; a feature architecture review may combine the best-practices and feature-diagram skills.

## Install in Codex

Ask Codex to install one or more skill directories from this repository. For the complete collection:

```text
$skill-installer install these skills from big-yellow-duck/sveltekit-project-skills:
- skills/sveltekit-best-practices
- skills/portable-svelte-components
```

Or install only the skill needed for a project:

```text
$skill-installer install skills/sveltekit-best-practices from https://github.com/big-yellow-duck/sveltekit-project-skills
```

Installed skills become available on the next Codex turn. Invoke one explicitly with its `$skill-name`, select it from `/skills`, or allow Codex to trigger it from the task description.

## Repository-specific rules

These skills provide reusable defaults. A target repository's `AGENTS.md`, local skill instructions, configured framework version, scripts, and established conventions remain authoritative. The skills should adapt to the project rather than impose example directories or tools mechanically.

## Contributing

Keep each skill narrowly triggered and put detailed guidance in references. Add a separate skill when a workflow has a meaningfully different trigger or procedure. Validate every skill directory with Codex's `quick_validate.py` before publishing.
