# skills — AGENTS.md

## Repo structure

```
skills/       # Skill definitions (one kebab-case subdirectory per skill)
├── hardware-discovery/
├── skill-creator/       # Forked from Anthropic; adds eval cleanup
├── template-skill/      # Scaffold for new skills
├── git-sync/
├── type-hinting/        # Has references/{python,javascript}.md
eval/         # Evaluation scenarios (empty)
.opencode/    # Plugin config (gitignored at root)
```

## Key facts

- **No build, test, lint, or CI system.** Only tools are filesystem and agent tools.
- Skills follow the opencode protocol: each is a dir with a required `SKILL.md`, optional `scripts/`, and optional `references/`.
- `PROJECT_STRUCTURE_BLUEPRINT.md` documents full conventions — consult before restructuring.
- `.opencode/` (gitignored) has `@opencode-ai/plugin@1.17.11`. Run `npm install` inside it if you need node_modules locally.

## Adding a skill

```bash
cp -r skills/template-skill/ skills/<your-kebab-skill>/
```
Edit `SKILL.md`. Add helpers to `scripts/` or `references/` as needed.

## Style conventions

- Skill dir names: `kebab-case`
- Flat at `skills/<name>/` — no nesting
- No shared utilities between skills; keep them standalone
- `SKILL.md` frontmatter (name, description) strongly recommended for trigger detection
