# SvelteKit project skills

A reusable collection of focused Codex skills for keeping SvelteKit projects consistent without forcing every task through one oversized prompt.

## Skills

| Skill | Use it for |
| --- | --- |
| `sveltekit-project-structure` | Project trees, ownership, runtime boundaries, routes, server code, databases, services, and dependency direction |
| `svelte-component-development` | Everyday Svelte 5 components, runes, SSR, accessibility, forms, interaction, and component review |
| `portable-svelte-components` | Figma/design-to-code work and reusable, data-driven component families |

The skills are intentionally composable. A design-to-code task may use all three; a hydration fix should normally use only component development; an architecture review should normally use only project structure.

## Install in Codex

Ask Codex to install one or more skill directories from this repository. For the complete collection:

```text
$skill-installer install these skills from big-yellow-duck/sveltekit-project-skills:
- skills/sveltekit-project-structure
- skills/svelte-component-development
- skills/portable-svelte-components
```

Or install only the skill needed for a project:

```text
$skill-installer install skills/svelte-component-development from https://github.com/big-yellow-duck/sveltekit-project-skills
```

Installed skills become available on the next Codex turn. Invoke one explicitly with its `$skill-name`, select it from `/skills`, or allow Codex to trigger it from the task description.

## Repository-specific rules

These skills provide reusable defaults. A target repository's `AGENTS.md`, local skill instructions, configured framework version, scripts, and established conventions remain authoritative. The skills should adapt to the project rather than impose example directories or tools mechanically.

## Contributing

Keep each skill narrowly triggered and put detailed guidance in references. Add a separate skill when a workflow has a meaningfully different trigger or procedure. Validate every skill directory with Codex's `quick_validate.py` before publishing.
