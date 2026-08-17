## Sveltekit project skills
A collection of sveltekit project skills to make coding agents follow the a standardzied structure.

### Install in Codex

Ask Codex to install the skill from this repository:

```text
$skill-installer install the sveltekit-project-structure skill from https://github.com/big-yellow-duck/sveltekit-project-skills
```

### other agents 
idk just ask them to git clone and install it for you.

Codex should detect the installed skill automatically. If it does not appear, restart Codex. You can then select it with `/skills`, mention it as `$sveltekit-project-structure`, or let Codex invoke it automatically when a task matches its description.

### motivation
I've had many issues of agent implementing features that dont align with the current code base style.
The naive way before:

- explaining the coding conventions before implementing a feature. 
- long discussion with the agent to implement features
- interrupting mid way when the forgetting to mention the styles

### Rambling
I find it hard to jump around project without a consistent way of implementing so its serves as a guiding star to implement in a consistent structure across multiple projects.
makes reviewing and context switching between project less painful. 
