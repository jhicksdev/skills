# Agent Skills

A personal collection of agent skills — modular packages that extend AI agent capabilities.

## Structure

```
skills/
├── skills/           # Skill definitions (one directory per skill)
├── eval/             # Evaluation scenarios
├── .opencode/        # Agent configuration
└── README.md
```

## Usage

Each skill lives in its own directory under `skills/` with a `SKILL.md` file defining its behavior and triggers.

To add a new skill, copy `skills/template-skill/` and customize the `SKILL.md`.

## Requirements

- [opencode](https://opencode.ai) or compatible AI agent that supports the skills protocol
