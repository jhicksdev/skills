# skills — AGENTS.md

## Repo structure

```
skills/       # Skill definitions (one kebab-case subdirectory per skill)
├── hardware-discovery/
├── template-skill/     # Scaffold for new skills
eval/         # Evaluation scenarios (🗸 empty)
.opencode/    # Agent config (🗸 gitignored at root)
```

## Key facts

- **No build, test, lint, or CI system.** Only tools are standard filesystem and agent tools.
- Skills use the opencode skills protocol: each skill is a directory with a required `SKILL.md` and optional `scripts/`.
- `PROJECT_STRUCTURE_BLUEPRINT.md` documents the conventions in detail — consult it before restructuring.
- The `.opencode/` plugin dependency (`@opencode-ai/plugin@1.17.11`) is gitignored. Run `npm install` inside `.opencode/` if you need the plugin node_modules locally.

## Adding a skill

```bash
cp -r skills/template-skill/ skills/<your-kebab-skill>/
```

Then edit `SKILL.md`. Add helper scripts to `scripts/` if needed.

## Style conventions

- Skill directory names: `kebab-case`
- Each skill is flat at `skills/<name>/` — no nesting
- No shared utilities between skills; keep them standalone
- SKILL.md frontmatter (name, description) is strongly recommended for trigger detection
