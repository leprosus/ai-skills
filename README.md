# Go skills for Codex

This repository contains two independent Codex skills:

- `golang-cli` — Go command-line interfaces built with the standard library `flag` package.
- `golang-code-style` — Go code style preferences, with additional guidance in `references/details.md`.

## Use globally

Keep this repository as the editable source. Link each skill directory into the user-level Codex skills directory:

```sh
cd /absolute/path/to/ai-skills
mkdir -p "$HOME/.agents/skills"
ln -s "$PWD/skills/golang-cli" "$HOME/.agents/skills/golang-cli"
ln -s "$PWD/skills/golang-code-style" "$HOME/.agents/skills/golang-code-style"
```

The destination names must be free before creating the links. Codex follows these links, so edits in this checkout are available globally without copying the skills again. Codex normally detects local skill changes automatically; restart it if an update does not appear.

To bring in changes from GitHub, run `git pull --ff-only` in this repository. The links then point to the updated files. This repository does not run `git pull` automatically.

## Skill layout

Each skill has its own `SKILL.md` with a name and description. Supporting files stay beside that file and are read only when the skill calls for them.
