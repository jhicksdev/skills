---
name: git-sync
description: >-
  Commit changes and push to remote in one workflow, with conventional commit
  message analysis, intelligent staging, and message generation. Use when the
  user asks to commit and push, save and sync, or says "/commit". This skill
  handles the full pipeline: analyzing changes, staging intentionally, generating
  a conventional-commit message from the diff, committing, and pushing. Supports
  optional type/scope/description overrides. Do NOT use for git operations
  without a push step — use the git-commit skill instead.
---

# Git Sync — Commit + Push Workflow

## Overview

Stage, commit with Conventional Commits, and push to remote in a single workflow. Start by understanding the current state, then stage intentionally, generate an accurate commit message from the actual diff, commit, and push.

## Conventional Commit Format

```
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

## Commit Types

| Type       | Purpose                        |
|------------|--------------------------------|
| `feat`     | New feature                    |
| `fix`      | Bug fix                        |
| `docs`     | Documentation only             |
| `style`    | Formatting/style (no logic)    |
| `refactor` | Code refactor (no feature/fix) |
| `perf`     | Performance improvement        |
| `test`     | Add/update tests               |
| `build`    | Build system/dependencies      |
| `ci`       | CI/config changes              |
| `chore`    | Maintenance/misc               |
| `revert`   | Revert commit                  |

## Breaking Changes

```
# Exclamation mark after type/scope
feat!: remove deprecated endpoint

# BREAKING CHANGE footer
feat: allow config to extend other configs

BREAKING CHANGE: `extends` key behavior changed
```

## Workflow

### 1. Understand Context

```bash
git status --porcelain
git diff --stat
git log --oneline -5
```

Check whether changes are already staged, what areas they touch, and what branch you're on.

### 2. Review Changes

Read the actual diffs:

```bash
# Staged changes
git diff --staged

# Unstaged working changes
git diff
```

### 3. Stage Files Intentionally

Stage in logical groups. Never stage generated, temporary, or secret files.

```bash
# Stage specific files
git add path/to/file1 path/to/file2

# Stage by pattern
git add src/components/*
git add *.test.*

# Stage all tracked changes (careful — review first)
git add -u
```

**Never commit secrets** (.env, credentials.json, private keys, .env.local).

### 4. Generate Commit Message

Analyze the staged diff to determine:

- **Type**: What kind of change is this? Match against the conventional commit types.
- **Scope**: What area/module is affected? Use the component/package/directory name.
- **Description**: One-line summary in present tense, imperative mood, under 72 chars.

Scope examples: `feat(auth)`, `fix(api)`, `docs(readme)`, `refactor(parser)`.

For changes with no clear component scope, omit the scope: `chore: bump dependencies`.

### 5. Execute Commit

```bash
# Single line (most common)
git commit -m "<type>(<scope>): <description>"

# Multi-line with body/footer
git commit -m "$(cat <<'EOF'
<type>(<scope>): <description>

<optional body explaining why>

Closes #<issue>
EOF
)"
```

### 6. Push to Remote

After a successful commit:

```bash
git push
```

If the branch isn't tracking a remote:

```bash
git push -u origin HEAD
```

### 7. Verify

Confirm the push succeeded:

```bash
git status --porcelain
git log --oneline -3
```

## Overriding Type/Scope/Description

If the user specifies a type, scope, description, or any part of the commit message, use their input instead of auto-detecting. When the user provides an explicit instruction ("commit this as a fix"), honor it — they know their intent better than diff analysis.

## Git Safety Protocol

- NEVER update git config
- NEVER force push (`--force`, `--force-with-lease`) unless explicitly asked
- NEVER skip hooks (`--no-verify`) unless explicitly asked
- NEVER amend a commit that was already pushed
- NEVER commit or push secrets
- NEVER force push to main/master or the default branch
- NEVER amend commits (it rewrites history) — create a new commit instead
- If a commit fails due to hooks, fix the issue and create a NEW commit
- Only commit and push when explicitly requested by the user
